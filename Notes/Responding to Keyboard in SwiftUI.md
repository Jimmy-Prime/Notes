# Responding to Keyboard in SwiftUI

tags: SwiftUI, Keyboard

Dealing with keyboards is always so hard in iOS development. In UIKit programming, we have to deal with AutoLayout, Notifications, Animation, and lots of other stuff. This post will introduce a cleaner way to handle keyboards in SwiftUI.

Let's say we have a view with input fields:

``` Swift
struct SomeViewWithInputFields: View {
    var body: some View {
        ...
    }
}
```

When keyboard is shown, we want to append a Spacer below our content with the size of keyboard. We can start by inplementing a `KeyboardObserver` that handles keyboard notifications and propagates notification values.

``` Swift
class KeyboardObserver: ObservableObject {
    struct KeyboardProperty {
        let height: CGFloat
        let duration: Double

        static var none: Self { .init(height: 0, duration: 0) }
    }

    @Published var dockedKeyboardProperty = KeyboardProperty.none

    private var willChangeFrame: AnyCancellable?

    init() {
        willChangeFrame = NotificationCenter.Publisher(center: .default, name: UIResponder.keyboardWillChangeFrameNotification)
            .map {
                guard let userInfo = $0.userInfo,
                    let duration = userInfo[UIResponder.keyboardAnimationDurationUserInfoKey] as? Double,
                    let rect = userInfo[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else {
                    return .none
                }

                if rect.maxY == UIScreen.main.bounds.height { // 1
                    return .init(height: rect.height, duration: duration)
                } else {
                    return .init(height: 0, duration: duration)
                }
            }
            .assign(to: \.dockedKeyboardProperty, on: self)
    }

    deinit {
        willChangeFrame?.cancel()
    }
}
```

Notice at (1), we only handle docked keyboard, because when keyboard is floating, user can move the keyboard around, and we should not pad our content.

Instead of directly use `KeyboardObserver`, we can wrap Spacer and logic and `KeyboardObserver` into another View.

``` Swift
struct KeyboardSpacer: View {
    @EnvironmentObject var keyboardObserver: KeyboardObserver

    var body: some View {
        Spacer()
            .frame(height: keyboardObserver.dockedKeyboardProperty.height)
            .animation(.easeInOut(duration: keyboardObserver.dockedKeyboardProperty.duration))
    }
}
```

Now we can modify our view, wrap original content and `KeyboardSpacer` into `VStack`

``` Swift
struct SomeViewWithInput: View {
    var body: some View {
        VStack {
            ...
            KeyboardSpacer()
        }
    }
}
```

Finally, just remember to `.environmentObject(KeyboardObserver())` some where in the view tree.


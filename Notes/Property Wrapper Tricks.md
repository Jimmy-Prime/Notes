# PropertyWrapper Tricks

tags: Swift, Property Wrapper

最近 app 想要支援多帳號登入，很多資料需要重新規劃儲存方式。其中想分享的是存在 UserDefaults 的資料要怎麼重新定義才不需要改太多程式碼。

假設我們正在做聊天軟體，有一個使用者設定是要不要上傳原始品質的圖片，如果是連到公司站，就保留原始品質；如果連到自己的伺服器，就壓縮再上傳避免把硬碟塞爆。現在這個 Bool 設定直接存在 UserDefaults (這裡用到的 @UserDefault 和 [SE-0258 Property Wrapper](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md#user-defaults) 裡面舉的例子是一樣的，有興趣的話可以點進去看 proposal)

``` Swift
enum UserSettings {
    @UserDefault(key: "preserve_image_quality", defaultValue: false)
    static var preserveImageQuality: Bool
}
```

現在這個 Bool 每個登入的帳號都必須存一個，如果又想要保留 UserSettings.preserveImageQuality 的呼叫方式要怎麼做呢？

因為 UserDefaults 是一個 key/value pair 的結構，只要我可以幫每個使用者產生出獨一無二的 key，我就可以把這些東西全部存到 UserDefaults 裡面了。我們可以把使用者的識別碼串到 key 裡面

``` Swift
"\(Defaults.currentAccount.id)_preserve_image_quality" // 新的 key
```

這個新的 key pattern 可能需要用在很多 UserSettings 裡面，我們一樣用 Property Wrapper 來避免重複 boilerplate code

``` Swift
@propertyWrapper
struct AccountDefault<T> {
    let key: String
    let defaultValue: T
    private var defaultsKey: String {
        "\(Defaults.currentAccount.id)_\(key)"
    }
    var wrappedValue: T {
        get {
            UserDefaults.standard.object(forKey: defaultsKey) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: defaultsKey)
        }
    }
}
```

利用一個 computed property defaultsKey 讓每次 get/set 拿到的 key 都是目前的登入帳號串成的。現在 UserSettings 裡面就可以用 AccountDefault 改寫了。

``` Swift
enum UserSettings {
    @AccountDefault(key: "preserve_image_quality", defaultValue: false)
    static var preserveImageQuality: Bool
}
```

唯一改動的地方就是把 UserDefault 改成 AccountDefault。

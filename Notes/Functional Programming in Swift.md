# Functional Programming in Swift

tags: Functional Programming, Swift

Other than OOP or POP, Swift has some functional programming ability, let's explore some.

Let's say we are working on a image processing framework. We have a bunch of filters that operate on a `Image` type.

``` Swift
func blur(radius: CGFloat = 1, _ image: Image) -> Image { ... }

func erosion(strength: CGFloat = 1, _ image: Image) -> Image { ... }

func reduceNoise(level: CGFloat = 1, _ image: Image) -> Image { ... }
```

We process an image with some filters, the code may looks like this:

``` Swift
let processedImage = reduceNoise(erosion(blur(image)))
```

If we want non default arguments, the code becomes:

``` Swift
let processedImage = reduceNoise(level: 0.2, erosion(strength: 3, blur(radius: 5, image)))
```

This code is not easy to read, and we can not tell what it's trying to do at a glance. Currently this code has some problems:

1. Functions taking multiple arguments make it hard to modify. It is not so easy to reorder the filters.
2. Order of the written code is reverse to operating order. Code is written left to right, but operation occurs from inner to outer.

To solve the first problem, we can use a technique called Currying to rewrite filter functions.

From [Wikipedia](https://en.wikipedia.org/wiki/Currying):

> In mathematics and computer science, currying is the technique of translating the evaluation of a function that takes multiple arguments into evaluating a sequence of functions, each with a single argument.

Now we can rewrite curried version of filter functions:

``` Swift
func blur(radius: CGFloat = 1) -> (Image) -> Image { ... }

func erosion(strength: CGFloat = 1) -> (Image) -> Image { ... }

func reduceNoise(level: CGFloat = 1) -> (Image) -> Image { ... }
```

The image processing code can be rewritten as:

``` Swift
let processedImage = reduceNoise(level: 0.2)(erosion(strength: 3)(blur(radius: 5)(image)))
```

A bit better, but not enough improvement. To solve the second problem, we are finally getting into the functional part of Swift. We can define an operator the reverse function call, from `f(x)` to `x operator f`

``` Swift
precedencegroup PipeForward {
    associativity: left
}

infix operator |>: PipeForward

func |> <I, O>(x: I, f: (I) -> O) -> O {
    f(x) // x |> f
}
```

Now the code becomes:

``` Swift
let processedImage = image
    |> blur(radius: 3)
    |> erosion(strength: 3)
    |> reduceNoise(level: 0.2)
```

Intent of this code is very clear and very readable.

If this series of filters were repeated alot, we don't want to repeat everytime, how can we reuse this code?

From a imperative programming world, we might define another function to do the job.

``` Swift
func myFilter() -> (Image) -> Image {
    { $0 |> blur(radius: 3) |> erosion(strength: 3) |> reduceNoise(level: 0.2) }
}

let processedImage = image |> myFilter()
```

But in functional world, function is no other than variables, we can define a function that represent a combination of functions. We can do this in Swift by introducing another operator.

``` Swift
precedencegroup Composition {
    associativity: left
}

infix operator >>>: Composition

func >>> <I, M, O>(f: @escaping (I) -> M, g: @escaping (M) -> O) -> (I) -> O {
    { $0 |> f |> g } // this is equivalent to g(f($0))
}
```

So, we can `myFilter` as:

``` Swift
let myFilter = blur(radius: 3) >>> erosion(strength: 3) >>> reduceNoise(level: 0.2)
let processedImage = image |> myFilter
```

That's it, nice and clean.
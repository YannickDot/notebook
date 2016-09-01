# Swift lang

### Custom operators
One nice feature of the swift language is that we can create our own custom operators ans set its precedence against existing/built-in operators

The syntax is :
```swift
infix operator |> { associativity left precedence 160 }

public func <MY_OPERATOR> <T, U> (left:T, right:T -> U) -> U {
    return right(left)
}
```

Let's say we want to make Swift a little more functional by mimicking the |> (pipe) operator from F#/OCaml/Elm

```swift
infix operator |> { associativity left precedence 160 }

public func <MY_OPERATOR> <T, U> (left:T, right:T -> U) -> U {
    return right(left)
}
```

Now we want to create curried versions of map and filter that runs on Arrays :
```swift

func map <T,U> (fn: T -> U) -> (data: [T]) -> [U] {
    return {(data: [T]) -> [U] in
        var res:[U] = []
        for it in data {
            res.append(fn(it))
        }
        return res
    }
}

func filter <T> (predicate: T -> Bool) -> (data: [T]) -> [T] {
    return {(data: [T]) -> [T] in
        var res:[T] = []
        for it in data {
            if(predicate(it)) {
                res.append(it)
            }
        }
        return res
    }
}
```

Our operator is now functional!

```swift
func plusTwo(x:Int) -> Int {
    return x + 2
}

func multByTwo(x:Int) -> Int {
    return x * 2
}

func isOver(bound:Int) -> (Int) -> Bool {
    return { x in
        x > bound
    }
}

var n:[Int] = [1,2,3,4,5,6,7,8,9,10]

n |> map(plusTwo)
  |> map(multByTwo)
  |> filter(isOver(12))

// --> [14, 16, 18, 20, 22, 24]
```

```swift
n |> map(plusTwo)
  |> map(multByTwo)
  |> filter(isOver(12))
```

is equivalent to :

```swift
filter(isOver(12))(
  map(multByTwo)(
    map(plusTwo)(
      n
    )
  )
)
```

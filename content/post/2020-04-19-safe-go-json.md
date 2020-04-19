+++
tags = []
categories = []
date="2020-04-19T09:48:11+02:00"
title="Making Go JSON safe (for JavaScript)"
+++

Go has many things going for it. The marshalling package is generally one of them. Very well thought out and easy to use.

This package holds a surprise, though, when shipping JSON to software built with other languages: nil Go slices marshal to `null` rather than the empty array `[]`, and nil maps marshal to `null` rather than the empty map `{}`.

These are not bugs. Indeed, in Go, `nil` is the zero value for slices as well as for maps. They're reference types, pointer-like, and `null` is the right choice to encode `nil`. \
If you're shipping JSON back and forth between two Go programs, the `null`s for empty slices or maps are perfectly reasonable.

However, those `null`s wreak havoc on, say, your JavaScript frontend code expecting an empty array. It's not JS's responsibility to know that slices and maps are reference types in Go.

My team at work was stumped by this. Now and then, a `null` array would break our JS. We tried a few approaches, which were complex and incomplete.

However, there is a very simple solution: we can define a custom marshaler for any type we're interested in marshalling for JavaScript.

So let's say we have a `type Product struct {...}` and we want to marshal a slice of `Product` safely.

1. Define a type for the slice: `type Products []Product`
1. Write a custom marshaler for the new type (`Products`), like so:

``` go
    func (ps Products) MarshalJSON() ([]byte, error) {
        if len(ps) == 0 {
            empty := make([]Product, 0)
            return json.Marshal(empty)
        }
        return json.Marshal([]Product(ps))
    }
```

Note that the marshaler for `Products` dispatches to the default marshaler for slices of `Product`.  If you did `json.Marshal(ps)` you'd have an infinite loop!

You can verify:

``` go
var safeEmpty Products
safeJSON, _ := json.Marshal(safeEmpty)
fmt.Println(string(safeJSON)) // => []
```

For maps, the story is exactly the same:

1. Define a type for your map: `type Phonebook map[string]Client`
1. Write a custom `func (pb Phonebook) MarshalJSON() ([]byte, error)` that handles `nil` separately

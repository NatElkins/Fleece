Fleece
======

Fleece is a JSON mapper for F#. It simplifies mapping from [System.Json](http://bit.ly/1axIBoA)'s JsonValue onto your types, and mapping from your types onto JsonValue.
Its design is strongly influenced by Haskell's [Aeson](http://hackage.haskell.org/package/aeson-0.7.0.0/docs/Data-Aeson.html). Like Aeson, Fleece is designed around two typeclasses (in [FsControl](https://github.com/gmpl/FsControl) style) ToJSON and FromJSON.

###Example

For example, given this data type:

```fsharp
type Person = {
    Name: string
    Age: int
    Children: Person list
}
```

You can map it to JSON like this:

```fsharp
open System.Json
open Fleece
open Fleece.Operators

type Person with
    static member instance (ToJSON, x: Person, _:JsonValue) = fun () ->
        jobj [ 
            "name" .= x.Name
            "age" .= x.Age
            "children" .= x.Children
        ]

```

And you can map it from JSON like this:

```fsharp
type Person with
    static member instance (FromJSON, _: Person, _: Person ParseResult) = 
        function
        | JObject o ->
            let name = o .@ "name"
            let age = o .@ "age"
            let children = o .@ "children"
            match name, age, children with
            | Success name, Success age, Success children -> 
                Success {
                    Person.Name = name
                    Age = age
                    Children = children
                }
            | x -> Failure (sprintf "Error parsing person: %A" x)
        | x -> Failure (sprintf "Expected person, found %A" x)
```

Though it's much easier to do this in a monadic or applicative way. For example, using [FSharpPlus](https://github.com/gmpl/FSharpPlus):

```fsharp
open FSharpPlus

type Person with
    static member Create name age children = { Person.Name = name; Age = age; Children = children }

    static member instance (FromJSON, _: Person, _: Person ParseResult) = 
        function
        | JObject o -> Person.Create <!> (o .@ "name") <*> (o .@ "age") <*> (o .@ "children")
        | x -> Failure (sprintf "Expected person, found %A" x)

```

Or monadically:


```fsharp
type Person with
    static member instance (FromJSON, _: Person, _: Person ParseResult) = 
        function
        | JObject o -> 
            monad {
                let! name = o .@ "name"
                let! age = o .@ "age"
                let! children = o .@ "children"
                return {
                    Person.Name = name
                    Age = age
                    Children = children
                }
            }
        | x -> Failure (sprintf "Expected person, found %A" x)
```

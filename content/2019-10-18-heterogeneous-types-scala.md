+++
title = "Supporting heterogeneous types in Scala"
date = 2019-10-18
path = "blog/2019/10/18/heterogeneous-types-scala/"
+++

# The objective

We want to create a function that takes a list of scala classes as input, and outputs a list of all elements, each element being encoded as json.

For encoding in json, we use [circe](https://circe.github.io/).

This function should be generic and works for any type we can find a [circe](https://circe.github.io/circe/codec.html) `Encoder` for it.

Our first try could be something like the following function:
```scala
import io.circe.{Encoder, Json}

def encode[A: Encoder](l: List[A]): List[Json] =
  l.map(Encoder[A].apply)
```

This `encode` function seems to work:
- to encode several Strings:
```scala
encode(List("hello", "world"))
// List[io.circe.Json] = List("hello", "world")
```
- to encode several integers:
```scala
encode(List(1, 2, 3))
// List[io.circe.Json] = List(1, 2, 3)
```

But this function cannot be used with a list of heterogeneous types:
```scala
encode(List(1, "hello", 3))
/// error: diverging implicit expansion for type io.circe.Encoder[Any]
```

Indeed the scala compiler first resolves the type parameter `A` by finding a common type for `String` and `Int`. In this scala version, it chooses `Any`.

Then the scala compiler tries to find an instance of `Encoder[Any]` and fails to do so.

<iframe height="280px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=lmeb8C8/1&theme=dark&layout=h74"></iframe>


# First solution: replacing type parameters with path dependent types

## Introducing `ToEncode` with a path dependent type to capture the type of each element

One solution is to force the scala compiler to capture the `Encoder` instance for each element, instead of using one generic `Encoder` for the whole list.

For that, we introduce a trait to capture the value and the `Encoder` instance:
```scala
trait ToEncode {
  type Value
  def value: Value
  def encoder: Encoder[Value]
}
```

Our `encode` function does not have any type parameter anymore as the type of each element is already captured in the instance of `ToEncode`:

```scala
def encode(l: List[ToEncode]): List[Json] =
  l.map(v => v.encoder.apply(v.value))
```

Let's define some helper functions to create `ToEncode` instances:

```scala
def stringToEncode(v: String)(implicit instance: Encoder[String]): ToEncode = new ToEncode {
  type Value = String
  def value = v
  def encoder = instance
}

def intToEncode(v: Int)(implicit instance: Encoder[Int]): ToEncode = new ToEncode {
  type Value = Int
  def value = v
  def encoder = instance
}
```

Now we can use `encode` with homogeneous or heterogeneous types:
```scala
encode(List(stringToEncode("hello"), stringToEncode("world")))
// List[io.circe.Json] = List("hello", "world")

encode(List(intToEncode(1), intToEncode(2), intToEncode(3)))
// List[io.circe.Json] = List(1, 2, 3)

encode(List(intToEncode(1), stringToEncode("hello"), intToEncode(3)))
// List[io.circe.Json] = List(1, "hello", 3)
```

By making our helpers `implicit`, we can avoid some boilerplate:

```scala
encode(List("hello", "world"))
// List[io.circe.Json] = List("hello", "world")

encode(List(1, 2, 3))
// List[io.circe.Json] = List(1, 2, 3)

encode(List(1, "hello", 3))
// List[io.circe.Json] = List(1, "hello", 3)
```
<iframe height="625px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=jfGglKO/1&theme=dark&layout=h74"></iframe>

## Generic solution

Instead of defining helper functions like `stringToEncode` or `intToEncode` for each type we need, we can also have a generic solution by building a `ToEncode` whenever we find an `Encoder` instance:

```scala
trait ToEncode {
  type Value
  def value: Value
  def encoder: Encoder[Value]
}

object ToEncode {
  implicit def fromEncoder[A: Encoder](a: A): ToEncode = new ToEncode {
    type Value = A
    def value = a
    def encoder = Encoder[A]
  }
}

def encode(l: List[ToEncode]): List[Json] =
  l.map(v => v.encoder.apply(v.value))
```

Now we can use our `encode` function with any kind of type that has an `Encoder` instance.

```scala
encode(List("hello", "world"))
// List[io.circe.Json] = List("hello", "world")

encode(List(1, 2))
// List[io.circe.Json] = List(1, 2)

encode(List(1, "hello", 3))
// List[io.circe.Json] = List(1, "hello", 3)
```

<iframe height="550px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=IMTlBrw/2&theme=dark&layout=h74"></iframe>

## Disadvantage

To use this approach, we need one instance of `ToEncode` for each element in the list.

Calling `encode` with a list of 500 strings will create 500 instances of `ToEncode`!

# Second solution: using value classes to encode each element

After great feedback from [Travis](https://twitter.com/travisbrown/status/1186062469850112002), I could get rid of the `ToEncode` instances.

Let's introduce a value class that will contain the `Json` instance for each element:

```scala
class AsJson(val json: Json) extends AnyVal
```

We use a value class to (hopefully) avoid runtime instances. A version with Dotty could use phantom type.

Instead of encoding each value in the `encode` function, we encode directly when converting each element to a `AsJson`:

```scala
object AsJson {
  implicit def toAsJson[A: Encoder](a: A): AsJson = new AsJson(Encoder[A].apply(a))
}
```

The `encode` function has almost nothing to do except forcing the compiler to encode each element:

```scala
def encode(l: List[AsJson]): List[Json] =
  l.map(_.json)
```

By making `AsJson.toAsJson` visible in the scope, we can now use our `encode` function with heterogeneous types:

```scala
import AsJson._

encode(List("hello", "world"))
// List[io.circe.Json] = List("hello", "world")

encode(List(1, 2))
// List[io.circe.Json] = List(1, 2)

encode(List(1, "hello", 3))
// List[io.circe.Json] = List(1, "hello", 3)
```

<iframe height="400px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=XcTtE5u/1&theme=dark&layout=h74"></iframe>


# Feedback welcome

If you know a better solution, please do not hesitate to tell [me](https://twitter.com/simon_yann/status/1185276156561506304).

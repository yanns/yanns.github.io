+++
title = "Sangria & Scala Futures, round 1"
date = 2020-12-24
path = "blog/2020/12/24/sangria-future-round-1/"
+++

# Sangria and `Future`

[Sangria](https://sangria-graphql.github.io/) is a Scala library implementing GraphQL on the server side.

To use it, one defines the GraphQL schema by defining types, fields, and, for each field, how to solve it.

When Oleg started this library, he used the Scala `Future` as main representation for asynchronous computation.

The main entry point is the `sangria.execution.Executor`. Here is a simplified version:

```scala
case class Executor[Ctx, Root](schema: Schema[Ctx, Root])(implicit executionContext: ExecutionContext) {
  def execute(queryAst: ast.Document)(implicit scheme: ExecutionScheme): scheme.Result[Ctx, marshaller.Node] = {...}
}
```

where `scheme.Result` is by default implemented as a `Future`.

```scala
  implicit object Default extends ExecutionScheme {
    type Result[Ctx, Res] = Future[Res]
  }
```

The resolution of the GraphQL query is delegated to `sangria.execution.Resolver`, which is using `Future` internally:

```scala
class Resolver[Ctx](queryAst: ast.Document)(implicit executionContext: ExecutionContext) {
  def processFinalResolve(resolve: Resolve): Future[(Vector[RegisteredError], marshaller.Node)] = {...}
}
```

# `Future` is not the only option

In the `Scala` world, `Future` is not the only option to work with asynchronous computations.

And at the time I'm writing this post, [cats-effect](https://typelevel.org/cats-effect/) 3.x is being developed, providing a set of very interesting features and performance improvements.

I've also used Sangria in application using the [Monix](https://monix.io/) `Task` instead of `Future`. The code integrating Sangria is converting back and forth between `Task` and `Future`. It's not a major issue, but still does not feel optimal.

I think that there is value for a library like Sangria to better support alternatives to `Future`.


# Supporting alternatives to `Future`

So I'm trying to enhance Sangria to be able to support any alternatives to `Future`.

By looking at the code, I see some usages of `future.map`, `future.sequence`, etc.

## Abstracting with a functional library

My first reflex is to use a functional library like `cats` and to make the code abstract by using type classes like [`Monad`](https://typelevel.org/cats/typeclasses/monad.html), [`Applicative`](https://typelevel.org/cats/typeclasses/applicative.html) and so on.

But Oleg wanted the Sangria library to have minimal dependencies. He has not used any functional library.

So I'll respect this decision and try another approach.

## Abstracting over `Future`.

To make the Sangria code not depending on `Future` directly, I'm introducing a trait that should abstract `Future` away:

```scala
trait Effect[F[_]] {
  def pure[A](a: A): F[A]
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

To stay compatible with `Future`, I also provide a default implicit implementation for it.
If my experiment goes well, at the end, this should be the only place where Sangria is using `Future`.

```scala
object Effect {
  implicit def FutureEffect(implicit ec: ExecutionContext): Effect[Future] =
    new Effect[Future] {
      override def pure[A](a: A): Future[A] = Future.successful(a)
      override def map[A, B](fa: Future[A])(f: A => B): Future[B] = fa.map(f)
    }
}
```

I know it would be better to first have `flatMap` and `unit` as `map` could be expressed in `flatMap` of `unit` but I don't care for now. The name is also not optimal.

My goal is first to check if I can quickly make Sangria using `Effect` instead of `Future`.
Once Sangria can compile with it, and if the current tests relying on `Future` pass, I can refine `Effect`.

So I'm starting using `Effect`. The compiler is calling me names.

But I am changing code step by step, and this is feeling good. I can remove `Future` completely from some classes:

```scala
-import scala.concurrent.{ExecutionContext, Future}

-  case class DeferredResult(deferred: Vector[Future[Vector[Defer]]], futureValue: Future[Result])
+  case class DeferredResult[F[_]: Effect](
+      deferred: Vector[F[Vector[Defer]]],
+      futureValue: F[Result])
       extends Resolve {
     def appendErrors(
         path: ExecutionPath,
         errors: Vector[Throwable],
-        position: Option[AstLocation]) =
+        position: Option[AstLocation]): DeferredResult[F] =
       if (errors.nonEmpty)
-        copy(futureValue = futureValue.map(_.appendErrors(path, errors, position)))
+        copy(futureValue = Effect[F]().map(futureValue)(_.appendErrors(path, errors, position)))
       else this
   }
```

The code is a bit more complex but still manageable:

```scala
-  def resolveValue(
+  def resolveValue[F[_]: Effect](
       path: ExecutionPath,
       astFields: Vector[ast.Field],
       tpe: OutputType[_],
@@ -1231,23 +1239,23 @@ class Resolver[Ctx](
             resolveSimpleListValue(simpleRes, path, optional, astFields.head.location)
           else {
             // this is very hot place, so resorting to mutability to minimize the footprint
-            val deferredBuilder = new VectorBuilder[Future[Vector[Defer]]]
-            val resultFutures = new VectorBuilder[Future[Result]]
+            val deferredBuilder = new VectorBuilder[F[Vector[Defer]]]
+            val resultFutures = new VectorBuilder[F[Result]]

             val resIt = res.iterator

             while (resIt.hasNext)
               resIt.next() match {
                 case r: Result =>
-                  resultFutures += Future.successful(r)
-                case dr: DeferredResult =>
+                  resultFutures += Effect[F]().pure(r)
+                case dr: DeferredResult[F] =>
                   resultFutures += dr.futureValue
                   deferredBuilder ++= dr.deferred
               }

             DeferredResult(
               deferred = deferredBuilder.result(),
-              futureValue = Future
+              futureValue = Effect[F]()
                 .sequence(resultFutures.result())
                 .map(resolveSimpleListValue(_, path, optional, astFields.head.location))
             )
```

As I abstract more and more functions on `Future`, the `Effect` abstraction is growing:

```scala
trait Effect[F[_]] {
  def pure[A](a: A): F[A]
  def failed[T](exception: Throwable): F[T]
  def map[A, B](fa: F[A])(f: A => B): F[B]
  def sequence[A](vector: Vector[F[A]]): F[Vector[A]]
  def recover[A, U >: A](fa: F[A])(pf: PartialFunction[Throwable, U]): F[U]
}
```

It's not beautiful but I don't consider this as an issue for now. Once everything compiles, I will take time to refine it and to reduce the surface of this abstraction.

## `Promise` on the way

But then I encounter some code using `Promise`, like:

```scala
  case class ChildDeferredContext[F[_]: Effect](promise: Promise[Vector[F[Vector[Defer]]]]) {
    def resolveDeferredResult(uc: Ctx, res: DeferredResult[F]): F[Result] = {
      promise.success(res.deferred)
      res.futureValue
    }
```

`Promise` is a [standard Scala class](https://www.scala-lang.org/api/current/scala/concurrent/Promise.html) that allows to build a `Future` based on side effects.
It's low level, and easy to make mistakes when using it.

I don't know why Oleg has decided to use it. I guess it was for performance optimizations to avoid mapping over `Future` everywhere. Something to check later.

But it's quite a shock. I was not expecting that.

This makes an abstraction over `Future` much more complicated now.

## What to do now

Encountering `Promise` is a new challenge.

I see two options from here:

### option 1 - abstracting over `Promise`

The first option is to extend the abstraction to also abstract over `Promise`.

Frankly I doubt this will lead to somewhere. The resulting code might be quite complex. And it might be much more difficult to provide alternative implementations for those abstractions.

### option 2 - removing `Promise`

The second option is to remove the usage of `Promise`. For that I first need to understand why it was used.

Removing `Promise` could lead to more functional code at the end. But I'm afraid it is a big change.

# Sangria 1 - 0 Yann

So I've tried to abstract over `Future` for sangria, but it was more complex than expected.

Time to stop this first round.

If you have some suggestions how to deal with `Promise`, I'm all hear on [Twitter](https://twitter.com/simon_yann) or [Mastodon](https://mastodon.partipirate.org/@yanns).

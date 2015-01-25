---
out: IO-Monad.html
---

<div class="floatingimage">
<img src="http://eed3si9n.com/images/openphoto-6987bw.jpg">
<div class="credit">Daniel Steger for openphoto.net</div>
</div>

### IO モナド

論文の後半を読むかわりに [Rúnar さん (@runarorama)](http://twitter.com/runarorama) の [Towards an Effect System in Scala, Part 2: IO Monad](http://apocalisp.wordpress.com/2011/12/19/towards-an-effect-system-in-scala-part-2-io-monad/) を読もう:

> While ST gives us guarantees that mutable memory is never shared, it says nothing about reading/writing files, throwing exceptions, opening network sockets, database connections, etc.

以下に [`ST`]($scalazBaseUrl$/effect/src/main/scala/scalaz/effect/ST.scala) の型クラスコントラクトをもう一度:

```scala
sealed trait ST[S, A] {
  private[effect] def apply(s: World[S]): (World[S], A)
}
```

そしてこれが `IO` の型クラスコントラクトだ:

```scala
sealed trait IO[A] {
  private[effect] def apply(rw: World[RealWorld]): Trampoline[(World[RealWorld], A)]
}
```

`Trampoline` の部分を無視すると、`IO` は `ST` の状態を `RealWorld` に固定したものに似ている。`ST` 同様に `IO` object 下にある関数を使って `IO` モナドを作ることができる。

```scala
scala> import scalaz._, Scalaz._, effect._, IO._
import scalaz._
import Scalaz._
import effect._
import IO._

scala> val action1 = for {
         _ <- putStrLn("Hello, world!")
       } yield ()
action1: scalaz.effect.IO[Unit] = scalaz.effect.IOFunctions\$\$anon\$4@149f6f65

scala> action1.unsafePerformIO
Hello, world!

```

以下が `IO` の下にある IO アクションだ:

```scala
  /** Reads a character from standard input. */
  def getChar: IO[Char] = ...
  /** Writes a character to standard output. */
  def putChar(c: Char): IO[Unit] = ...
  /** Writes a string to standard output. */
  def putStr(s: String): IO[Unit] = ...
  /** Writes a string to standard output, followed by a newline.*/
  def putStrLn(s: String): IO[Unit] = ...
  /** Reads a line of standard input. */
  def readLn: IO[String] = ...
  /** Write the given value to standard output. */
  def putOut[A](a: A): IO[Unit] = ...
  // Mutable variables in the IO monad
  def newIORef[A](a: => A): IO[IORef[A]] = ...
  /**Throw the given error in the IO monad. */
  def throwIO[A](e: Throwable): IO[A] = ...
  /** An IO action that does nothing. */
  val ioUnit: IO[Unit] = ...
}
```

`IO` object の `apply` メソッドを使って独自のアクションを作ることもできる:

```scala
scala> val action2 = IO {
         val source = scala.io.Source.fromFile("./README.md")
         source.getLines.toStream
       }
action2: scalaz.effect.IO[scala.collection.immutable.Stream[String]] = scalaz.effect.IOFunctions\$\$anon\$4@bab4387

scala> action2.unsafePerformIO.toList
res57: List[String] = List(# Scalaz, "", Scalaz is a Scala library for functional programming., "", It provides purely functional data structures to complement those from the Scala standard library., ...
```

TESS2:

> Composing these into programs is done monadically. So we can use `for`-comprehensions. Here’s a program that reads a line of input and prints it out again:

```scala
def program: IO[Unit] = for {
  line <- readLn
  _    <- putStrLn(line)
} yield ()
```


> `IO[Unit]` is an instance of `Monoid`, so we can re-use the monoid addition function `|+|`.

試してみよう:

```scala
scala> (program |+| program).unsafePerformIO
123
123

```

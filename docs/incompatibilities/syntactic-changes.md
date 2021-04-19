---
id: syntactic-changes
title: Syntactic Changes
---

Scala 3 introduces the optional-braces syntax and the new control structure syntax.
It comes at the cost of some minimal restrictions in the preexisting syntax.
Also the new Scala 3 features brings new restricted keywords.

Other changes tend to make the syntax less surprising and more consistent.

It is worth noting that most of the changes can be automatically handled during the [Scala 3 migration compilation](../tooling/scala-3-migration-mode.md).

||Scala 2.13|Scala 3 Migration Rewrite|Scalafix Rule|Comments|
|--- |--- |--- |--- |--- |
|[Restricted keywords](#restricted-keywords)||✅||The Scala 3 rule does not handle all cases|
|[Procedure syntax](#procedure-syntax)|Deprecation|✅|[✅](https://scalacenter.github.io/scalafix/docs/rules/ProcedureSyntax.html)||
|[Parentheses around lambda parameter](#parentheses-around-lambda-parameter)||✅|[✅](https://github.com/ohze/scala-rewrites/tree/dotty/#fixscala213parensaroundlambda)||
|[Open brace indentation for passing an argument](#open-brace-indentation-for-passing-an-argument)||✅|||
|[Wrong indentation](#wrong-indentation)||||Can be handled by a code formatting tool|
|[`_` as a type parameter](#--as-a-type-parameter)|||||
|[`+` and `-` as type parameters](#-and---as-type-parameters)|||||

## Restricted Keywords

The list of Scala 3 keywords can be found [here](https://dotty.epfl.ch/docs/internals/syntax.html#keywords).
_Regular_ keywords cannot be used as identifiers, whereas _soft_ keywords are not restricted.

For the matter of migrating from Scala 2.13 to Scala 3, only the subset of new _regular_ keywords are problematic.
It is composed of:
- `enum`
- `export`
- `given`
- `then`
- `=>>`
- `?=>`

For instance, the following piece of code compiles in Scala 2.13 but not in Scala 3.

```scala
object given { // Error: given is now a keyword, write `given` instead of given to keep it as an identifier
  val enum = ??? // Error: enum is now a keyword, write `enum` instead of given to keep it as an identifier

  println(enum) // Error: enum is now a keyword, write `enum` instead of given to keep it as an identifier
}
```

The [Scala 3 migration compilation](../tooling/scala-3-migration-mode.md) rewrites the code into:

```diff
-object given {
-  val enum = ???

-  println(enum)
+object `given` {
+  val `enum` = ???

+  println(`enum`)
}
```

## Procedure Syntax

Procedure syntax has been deprecated for a while and it is dropped in Scala 3.
The following pieces of code are now illegal:

```scala
object Bar {
  def print() { // Error: Procedure syntax no longer supported; `: Unit =` should be inserted here
    println("bar")
  }
}
```

The [Scala 3 migration compilation](../tooling/scala-3-migration-mode.md) rewrites the code into.

```diff
object Bar {
-  def print() {
+  def print(): Unit = {
    println("bar")
  }
}
```

## Parentheses Around Lambda Parameter

When followed by its type, the parameter of a lambda is now required to be enclosed in parentheses.
The following piece of code is invalid.

```scala
val f = { x: Int => x * x } // Error: parentheses are required around the parameter of a lambda
```

The [Scala 3 migration compilation](../tooling/scala-3-migration-mode.md) rewrites the code into:

```diff
-val f = { x: Int => x * x }
+val f = { (x: Int) => x * x }
```

## Open Brace Indentation For Passing An Argument

In Scala 2 it is possible to pass an argument after a new line by enclosing it into braces.
Although valid, this style of coding is not encouraged by the [Scala style guide](https://docs.scala-lang.org/style) and is no longer supported in Scala 3.

This syntax is now invalid:
```scala
test("my test")
{ // Error: This opening brace will start a new statement in Scala 3.
  assert(1 == 1)
}
```

The [Scala 3 migration compiler](../tooling/scala-3-migration-mode.md) indents the first line of the block.

```diff
test("my test")
-{
+  {
  assert(1 == 1)
}
```

This migration rule applies to other patterns as well, such as refining a type after a new line.

```diff
type Bar = Foo
- {
+   {
  def bar(): Int
}
```

A preferable solution is to write:

``` diff
-test("my test")
-{
+test("my test") {
  assert(1 == 1)
}
```

## Wrong indentation

The Scala 3 compiler now requires correct indentation.
The following pieces of code that compiled in Scala 2 does not compile anymore, because of the indentation.

```scala
def bar: (Int, Int) = {
  val foo = 1.0
  val bar = foo // [E050] Type Error: value foo does not take parameters
    (1, 1)
} // [E007] Type Mismatch Error: Found Unit, Required (Int, Int)
```

The indentation must be fixed.

```diff
def bar: (Int, Int) = {
  val foo = 1.0
  val bar = foo
-    (1, 1)
+  (1, 1)
}
```

These errors can be prevented by using a Scala formatting tool such as [scalafmt](https://scalameta.org/scalafmt/) or the [IntelliJ Scala formatter](https://www.jetbrains.com/help/idea/reformat-and-rearrange-code.html).
Beware that these tools may change the entire code style of your project.

## `_` As A Type Parameter

The usage of the `_` identifier as a type parameter is permitted in Scala 2.13, even if it has never been mentioned in the Scala 2 specification.
It is used in the API of [fastparse](https://index.scala-lang.org/lihaoyi/fastparse), in combination with a context bound, to declare an implicit parameter.

```sala
def foo[_: Foo]: Unit = ???
```

Here, the method `foo` takes a type paramater `_` and an implicit parameter of type `Foo[_]` where `_` refers to the type parameter, not the wildcard symbol.

Martin Odersky described this pattern as a "clever exploit of a scalac compiler bug" ([source](https://www.reddit.com/r/scala/comments/fczcvo/mysterious_context_bounds_in_fastparse_2/fjecokn/)).

The Scala 3 compiler does not permit this pattern anymore: 

```text
-- [E040] Syntax Error: src/main/scala/anonymous-type-param.scala:4:10
4 |  def foo[_: Foo]: Unit = ()
  |          ^
  |          an identifier expected, but '_' found
```

The solution is to give the parameter a valid identifier name, for instance `T`.
This will not break the binary compatibility.

```diff
-def foo[_: Foo]: Unit = ???
+def foo[T: Foo]: Unit = ???
```

## `+` And `-` As Type Parameters

`+` and `-` are not valid identifiers for type parameters in Scala 3, since they are reserved for variance annotation.

You cannot write `def foo[+]` or `def foo[-]` anymore.

```
-- Error: src/main/scala/type-param-identifier.scala:2:10 
2 |  def foo[+]: +
  |          ^
  |          no `+/-` variance annotation allowed here
```

The solution is to choose another valid identifier, for instance `T`.

However, `+` and `-` still are valid type identifiers in general.
You can write `type +`.

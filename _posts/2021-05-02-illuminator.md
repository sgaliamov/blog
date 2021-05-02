---
title: Generating code in runtime with Illuminator
categories: ["library", "msil"]
created: 2020-01-01
date: 2021-05-02
layout: post
---

Illuminator is yet another wrapper around [`ILGenerator`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.ilgenerator), but with some interesting features:

1. [Fluent, convenient API](#fluent-api) with functional programming flavor.
1. [Extensibility](#custom-extensions).
1. [Tracing generated code](#tracing).
1. [Transparent abstraction](#final-thoughts).
1. `.netstandard2.0` support.

This library was emerged from another project, which I implemented with code emission, and was field tested in it.\
It's a [library](https://github.com/sgaliamov/il-lighten-comparer) which can generate comparers on runtime for any structure or class.

## Fluent API

Let imagine we need to generate the following code:

``` c#
int Foo(int value) {
    if (value == 2) {
        return 1;
    }

    return value + 3;
}
```

Using vanilla `ILGenerator` you may write something like this:

``` c#
var method = new DynamicMethod("Foo", typeof(int), new[] { typeof(int) });
var generator = method.GetILGenerator();
var label = generator.DefineLabel();

generator.Emit(OpCodes.Ldarg_0);
generator.Emit(OpCodes.Ldc_I4_2);
generator.Emit(OpCodes.Ceq);
generator.Emit(OpCodes.Brfalse_S, label); // if (value == 2)
generator.Emit(OpCodes.Ldc_I4_1);
generator.Emit(OpCodes.Ret);              // return 1
generator.MarkLabel(label);
generator.Emit(OpCodes.Ldarg_0);
generator.Emit(OpCodes.Ldc_I4_3);
generator.Emit(OpCodes.Add);
generator.Emit(OpCodes.Ret);              // return value + 3

var foo = method.CreateDelegate<Func<int, int>>();
```

So much code for such simple function! When you need to write a more complex thing, it becomes not possible to maintain and understand it.

There are few problems with this code:

1. It's very verbose, repetitive and hard to read. All this `Emit` and `ObCodes` are just unnecessary noise.
1. It's very hard to write such code. You need to remember the specification for each instruction and keep in mind the state of the evaluation stack. You need to know exactly how many parameters each instruction needs.
1. It's error prone. It's very easy to make a mistake, and exceptions that are thrown at runtime are not very informative.

The simplest thing that we can do to improve the situation is to introduce "fluent" API:

``` c#
var method = new DynamicMethod("Foo", typeof(int), new[] { typeof(int) });
var generator = method.GetILGenerator();
var label = generator.DefineLabel();
var il = generator.UseIlluminator(); // Creates wrapper

il.Ldarg_0()
  .Ldc_I4_2()
  .Ceq()
  .Brfalse_S(label) // if (value == 2)
  .Ldc_I4_1()
  .Ret()            // return 1
  .MarkLabel(label)
  .Ldarg_0()
  .Ldc_I4_3()
  .Add()
  .Ret();           // return value + 3

var foo = method.CreateDelegate<Func<int, int>>();
```

Much better this time:

1. We don't have to memorise all `OpCodes` and write those endless `Emit`. Intellisense helps us.
1. It less verbose and much easier to read.

Still, this does not solve all problems.
It is still possible to have invalid evaluation stack or misuse short versions for branching instructions (`Brfalse_S`).

Lets try one more time with some functional helpers:

``` c#
using static Illuminator.Functions;
...
il.Brfalse_S(Ceq(Ldarg_0(), Ldc_I4_2()), out var label)  // if (value == 2) {
  .Ret(Ldc_I4_1())                                       //   return 1;
  .MarkLabel(label)                                      // }
  .Ret(Add(Ldarg_0(), Ldc_I4_3()));                      // return value + 3;
```

You may think: What?! Wait, try to read it again: it checks (`Brfalse_S`) the result of comparing (`Ceq`) the first argument (`Ldarg_0`) and the constant two (`Ldc_I4_2`); if they are equal, return 1 (`Ret(Ldc_I4_1())`), otherwise return the sum of the argument and three (`Ret(Add(Ldarg_0(), Ldc_I4_3()))`).

The code now is much shorter and close to the target `C#` version.
It's nicer to write such code, because all methods have exact amount of parameters that they need, and with output parameters we don't have to break "fluent flow" to create labels or locals (`out var label`).

## Hot does it work

Lets look at this line:

``` c#
Add(Ldarg_0(), Ldc_I4_3())
```

`Add` is the static function from the `Functions` class:

``` c#
public delegate ILEmitter ILEmitterFunc(in ILEmitter emitter);
...
public static ILEmitterFunc Add(ILEmitterFunc func1, ILEmitterFunc func2) =>
    (in ILEmitter il) => il.Add(func1, func2);
```

We can use it directly thanks to [using static directive](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-static) feature. That why we need to include `using static Illuminator.Functions;` at the beginning.

This function uses the extension for `ILEmitter` class which calls `ILEmitterFunc` functions before it calls the actual `Add` methods:

``` c#
public static ILEmitter Add(this ILEmitter self, in ILEmitterFunc func1, in ILEmitterFunc func2) {
    func1(self);
    func2(self);

    return self.Add(); // and this finally does the real generator.Emit(OpCodes.Add);
}
```

`ILEmitterFunc` functions are responsible for preparing values in the stack. And as you may guess, it can be much complex list of constructions than simple `Ldc_I4`.

As the result we get the fluent, convenient API with functional programming flavor:

1. It uses the original naming of `MSIL` instructions, so you don't need to guess what `Ldc_I4_2` for example does. \
   It does exactly `generator.Emit(OpCodes.Ldc_I4_2);`.
1. All methods have helpers with exact amount of `ILEmitterFunc` parameters that they need to execute.
1. We can use output parameters to not break fluent flow.
1. A flow of instructions can be reused many times as it is a first class function now.

## Who it is implemented

Writing all those functional extensions and wrappers manually is really boring and takes a lot of time.

To make things fun and make fewer mistakes I created a complementary [tool](https://github.com/sgaliamov/illuminator/tree/master/src/Illuminator.Cli) to generate most of the library's code using `F#`, `Scriban` library as a template engine, documentation and reflection as a basic information.

I've used [documentation](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.opcodes?view=net-5.0) to create [opcodes.json](https://github.com/sgaliamov/illuminator/blob/master/src/Illuminator.Cli/opcodes.json) file to define what exact parameters we need to run an instruction. So I can generate methods like this:

``` c#
public ILEmitter Beq_S(in Label label) {
    ValidateJump(label);                // can validate that short version of a jump instruction is used correctly.
    _il.Emit(OpCodes.Beq_S, label);     // pass the right parameter that the instruction needs.
    _logger?.Log(OpCodes.Beq_S, label); // embed the logger into each method.

    return this;
}
```

And using information in the `System.Reflection.Emit.OpCodes` class I know how much items an instruction are needed to generate functional extensions. For example `OpCodes.Beq_S` needs 2 values in the evaluation stack, to it creates the following methods:

``` c#
public static ILEmitter Beq_S(
    this ILEmitter self,
    in ILEmitterFunc func1,
    in ILEmitterFunc func2,
    in Label label) {
    func1(self);
    func2(self);

    return self.Beq_S(label);
}
...
public static ILEmitterFunc Beq_S(
    ILEmitterFunc func1, ILEmitterFunc func2, Label label) =>
    (in ILEmitter il) => il.Beq_S(func1, func2, label);
```

## Custom extensions

We can improve the readability of the code by creating our own extensions. Lets look at this one:

``` c#
namespace CustomExtensions
{
    public static class Functions
    {
        public static ILEmitterFunc If(
            ILEmitterFunc condition,
            ILEmitterFunc then,
            ILEmitterFunc @else) => (in ILEmitter il) =>
            il.Brfalse(condition, out var otherwise)
              .Emit(then)
              .Br(out var end)
              .MarkLabel(otherwise)
              .Emit(@else)
              .MarkLabel(end);
    }
}
```

This extension emits the equivalent of the following code:

``` c#
if (condition()) {
    then();
} else {
    otherwise();
}
```

Using this extension we can rewrite our previous attempt like this:

``` c#
using static CustomExtensions.Functions;
...
var foo = new DynamicMethod("Foo", typeof(int), new[] { typeof(int) })
    .GetILGenerator()
    .UseIlluminator(
        enableTraceLogger: true,             // enable tracing (optional)
        Ret(If(Ceq(Ldarg_0(), Ldc_I4_2()),   // condition
               Ldc_I4_1(),                   // then
               Add(Ldarg_0(), Ldc_I4_3())))) // else
    .CreateDelegate<Func<int, int>>();
```

At this time, the code is now even easier and more convenient for reading!

Of course, it can take a while for you to adapt to the functional style, but it doesn't have to.
You can still use fluent syntax.
In my code I usually mix fluent and functional approaches depending on a situation.

## Tracing

In complex cases you will want to see the list of instructions that your code generates.
To do it you can use `enableTraceLogger` parameter, you may notice it in the last example.

It uses `System.Diagnostics.Trace` and outputs such result:

``` vb
Int32 Foo(Int32)
          1: .ldarg.0         | 1 # we can see resulting the stack size
          2: .ldc.i4.2        | 2
          4: .ceq             | 1
          9: .brfalse Label_0 | 0 # we can see where the branching instruction is pointing
         10: .ldc.i4.1        | 1
         15: .br Label_1      | 0
    Label_0:                      # named label
         16: .ldarg.0         | 1
         17: .ldc.i4.3        | 2
         18: .add             | 1
    Label_1:
         19: .ret             | 0
```

## Final thoughts

The main goal of this project is to have simple, transparent library, which does exactly what you tell it.\
No overengineering, no custom names, no complex rules to follow.

Very often different libraries try to hide the underlying technology which they use.
For example, Entity Framework tries to hide SQL from programmers.
But it does a disservice eventually, because in order to write efficient code, you have to know EF itself, what SQL it generates, know all caveats, and understand SQL anyway, to be able to understand what your code really does.

Developers who generate code using `MSIL` have to know instructions.
Doing magical tweaks and hiding `Ldc_I4` for example behind nice `LoadInteger` name does not make things easier.
A programmer is forced to read the documentation or source code to find out what such methods actually do.

It does not mean that it's not allowed to create your own magical helpers.
You can and you should to make your own solution more maintainable.
But this is not the responsibility of the library.

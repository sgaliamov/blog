---
title: Rust, pros and cons
categories: ["reviews", "rust"]
created: 2020-10-23
layout: post
---

While working and reading The Book I collected things that I like and do not.
Here is my list. Of course, I may miss some important consents and be very subjective, but it's my list after all.

## Pros

1. **Immutability.**
1. **Performance.**
1. **Simple, minimalistic language.** I love C# but it growing so much last years.\
   I wish Microsoft will do something like JetBrains did with Kotlin.\
   Personally I think we need reboot the main `.NET`.
1. **Takes a lot from FP**: If you think that OOP is the only right way to design software, you are very wrong.\
   And if you persist in this, the fate of the dinosaurs awaits you.\
   You will have to maintain old legacy systems, like someone maintain COBOL code now, while rest of the world creates new shiny projects.
   1. shadowing.
   1. strong pattern matching.
   1. immutability. In C# we can define immutability using `readonly` keyword, and we have to define it on design time. with Rust you can decide on the moment whe you use it.
   1. expression returns value from the last line like in `F#`.
   1. `if` can be used as expression `let number = if condition { 5 } else { 6 };` like in `F#`. so no ternary operator.
   1. types inference. not so powerful like in F# or Haskell.
   1. enums are based on `algebraic data types`.
1. Effective **iterators**, with functionality  like in `C#`.
1. Build in **documentation**.
1. Build in **formatting** style.
1. Ownership concept for **memory management**. Similar with Resource Acquisition Is Initialization (RAII) from C++. No need for GC.
1. Very **strict compiler**. Rust is like C++ with best practices embedded in the compiler.
1. What makes code safe and enable **Zero cost abstraction**. Контроль корректности заимствований происходит во время компиляции и не порождает дополнительного исполнимого кода (принцип абстракций с нулевой стоимостью)
1. **No `nulls`!**
1. **No exceptions.**
1. **Generics**.
1. **Traits** better that interface.\
   Because you can extend any type without changing it.
   New traits can be implemented for existing types.
   You can define a trait for a whole type.
1. Build-in **test framework**. It's interesting idea to have tests mixed with the code. It allows write unit tests much faster. Gives 1re information about intentions. And gives access to internal implementation.
1. **Tree shaking?**
1. **async/.await!**
1. **Channels.**
1. arrow functions with **closures**.
1. Flexible **smart pointers**. But it is hard to master them.
1. `**Newtype**` feature is quite useful to create domain specific types.
1. Easy to read.
1. Learning curve is high. You have to read documentation first. Rust actually is a great protection against such developers that managed to learn only one ui framework and thinks they are programmers now. It's too high learning curve for them. So you can be sure that you are in the right company with rustafarians. Rust will protect your team from such coders that learned one language and one ui framework and think that they are programmers now.

## Cons

1. Learning curve is high. You have to read documentation first. Rust actually is a great protection against such developers that managed to learn only one ui framework and thinks they are programmers now. It's too high learning curve for them. So you can be sure that you are in the right company with rustafarians. Rust will protect your team from such coders that learned one language and one ui framework and think that they are programmers now.
   1. Very strict compiler. Hard to write code fast.
   2. Complex strings.
   3. Ownership concept for memory management. Have to understand some specific that you may not expect. The following code will not compile:

      ``` rust
      fn main() {
         let s1 = String::from("hello");
         let s2 = s1;
         println!("{}, world!", s1);
      }
      ```

      In C# the boxing is just a question on an interview.
      This knowledge does not have practical application most of the time.
      But in Rust you really use it and if you don't understand it, you will not be able to write code.

1. IDE support. Need some dancing to configure debugging.
   1. Autocompletion is not perfect. But it's not bad.
   1. Debugging works, but not so convenient like with C#. Optimizer is very aggressive and remove some variables.
1. No reflection. Things like de-serialization and IoC containers are not easy to implement, but possible. Adopting the `Poor Man's DI` approach can solve this problem.
1. No ternary operator. Very minor minus. But why not have it?
1. It's a very new language still. Some documentation is missing.
1. Snake case.
1. When deal with generics (collect) have to declare type and temp variable. Hard to chain in the same way how we chain generic methods in C#.
1. Memory fragmentation is possible. But it's not really important.
1. Rust doesn’t allow you to create your own operators or overload arbitrary operators.
1. Still can implement template method:

   ``` rust
   trait Base {
      fn next(&self) -> i32;
      fn foo(&self) -> i32 {
         *&self.next()
      }
   }

   trait Child: Base {
      fn next(&self) -> i32 {
         1
      }
   }
   ```

1. It's not clear when object is moved or copied. `let actual = foo(s);` it depends on whether `s` has `Copy` trait.
1. To get the most interesting things, like async closures, you have to use a nightly build. What is not safe in perspective.
1. Strange constructs like `PhantomData`.
1. Hard to do refactoring.\
   Basically you will have to rewrite your code.
1. Harder to use traits in methods.\
   Have to use `dyn` or define life scope explicitly.

## Summary

Implementing two sample projects does not make me expert.

Rust is a good language for system level developers who tired from C and C++.

Rust is not ready for big enterprise projects.
Java developers should stay with Java, and .NET developers will be more productive with C#.

Not all developers a "geeky" enough to deal with all this tunning and struggling with immaturity of the language.
Most of them are just normal people.

I think that languages with "ownership" is a new chapter in programming and we will see more languages like Rust in a future.

We definitely need better languages than we have now.
The complexity and monstrosity of mainstream languages push people to look for alternatives and create new languages like `Rust` or `Go`.
So in the near future I expect to see more languages with borrow checker.

Rust is a salvatore for those developers who want to jump into system programming but don't want to invest time into C++.
If you already formed developer, then switching from high level language to low level will make you junior for a year.
But with Rust we all more or less in equal conditions.
It's new growing ecosystem where it is not to late to jump into.

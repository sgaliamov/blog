# Rust Programming Language

named by the kind of fungi.
do not introduce ideas that appeared last 10 years. only old proven ideas.

## Pros

1. Immutability.
2. Performance.
3. Simple, minimalistic language.
4. Takes a lot from FP: If you think that OOP is the only right way to design software, you are very wrong. and if you persist in this, the fate of the dinosaurs awaits you. You will have to maintain old legacy systems, like someone maintain COBOL code now, while rest of the world creates new shiny projects.
   1. shadowing.
   2. strong pattern matching.
   3. immutability. In C# we can define immutability using `readonly` keyword, and we have to define it on design time. with Rust you can decide on the moment whe you use it.
   4. expression returns value from the last line like in `F#`.
   5. `if` can be used as expression `let number = if condition { 5 } else { 6 };` like in `F#`. so no ternary operator.
   6. types inference. not so powerful like in F# or Haskell.
   7. enums are based on `algebraic data types`.
5. Effective iterators, with functionality  like in `C#`.
6. Build in documentation.
7. Build in formatting style.
8. Ownership concept for memory management. Similar with Resource Acquisition Is Initialization (RAII) from C++. No need for GC.
9. Very strict compiler. What makes code safe and enable Zero cost abstraction.
10. No `nulls`!
11. No exceptions.
12. Generics.
13. Traits better that interface. Because you can extend any type without changing it. New traits can be implemented for existing types.
14. Build-in test framework. It's interesting idea to have tests mixed with the code. It allows write unit tests much faster. Gives more information about intentions. And gives access to internal implementation.
15. Tree shaking?
16. async/.await!
17. messaging for threads.
18. arrow functions with closures.
19. Flexible smart pointers. But it is hard to master them.
20. `Newtype` feature is quite useful to create domain specific types.

## Cons

1. Learning curve is high. You have to read documentation first.
   1. Very strict compiler. Hard to write code fast.
   2. Complex strings.
   3. Ownership concept for memory management. Have to understand some specific that you may not expect. The following code will not compile:
      ``` rs
      fn main() {
         let s1 = String::from("hello");
         let s2 = s1;
         println!("{}, world!", s1);
      }
      ```
2. IDE support. Need some dancing to configure debugging.
3. Autocompletion is not perfect. But it's not bad.
4. Debugging works, but not so convenient like with C#. Optimizer is very aggressive and remove some variables.
5. No reflection. Things like de-serialization and IoC containers are not easy to implement, but possible. Adopting the `Poor Man's DI` approach can solve this problem.
6. No ternary operator. Very minor minus. But why not have it?
7. It's a very new language still. Some documentation is missing.
8. Snake case.
9. When deal with generics (collect) have to declare type and temp variable. Hard to chain in the same way how we chain generic methods in C#.
10. Memory fragmentation is possible. But it's not really important.
11. Rust doesn’t allow you to create your own operators or overload arbitrary operators.
12. Still can implement template method:
      ``` rs
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
13. It's not clear when object is moved or copied. `let actual = foo(s);` it depends on whether `s` has `Copy` trait.
14. To get the most interesting things, like async closures, you have to use a nightly build. What is not safe in perspective.

In C# the boxing is just a question on an interview.
This knowledge does not have practical application most of the time.
But in Rust you really use it and if you don't understand it, you will not be able to write code.

Rust will protect your team from such coders that learned one language and one ui framework and think that they are programmers now.

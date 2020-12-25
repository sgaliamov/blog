---
created: 2020-10-23
tags: rust, programming languages, reviews, benchmarks, asp.net
---

# My first acquaintance with the Rust language

## Intro

This post is about my first practical experience of acquaintance with the language.
It can not be an exhaustive overview because I am not a professional `Rust` developer.
And of course I will not tell anything new about the language.
But I think people who are interested in `Rust` should keep the interest and promote it.

To make the post not completely boring, I did a benchmarking to compare implementations based on `r2d2`, `bb8` crates, and `ASP.NET`.

So, after ten years of development and five years after the [official release](https://blog.rust-lang.org/2015/05/15/Rust-1.0.html), `Rust` should be mature enough.
Right?
When it has been ranked as the ["The Most Loved Programming Language"](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-languages-loved) for five years in a row, you should definitely try it.
And I've tried.
I created two simple projects to learn it.

In my research, I discovered some facts about `Rust` that I found interesting.

First.
No one really knows why the language has such a name, and it is likely that even Graydon Hoare, the author of the language, [does not know it](https://www.reddit.com/r/rust/comments/27jvdt/internet_archaeology_the_definitive_endall_source/).
The working version is that it's named after the kind of a [fungi](https://en.wikipedia.org/wiki/Rust_%28fungus%29).

And second.
Rust may seem revolutionary to some people, but one of the creators of the language [said](https://tim.dreamwidth.org/1784423.html):
> ... we've tried hard to avoid incorporating new technology into it. We haven't always succeeded at failing to be novel, but we have a rule of thumb of not including any ideas in the language that are new as of the past ten years of programming language research...

And I find it very wise.
There are so many "hipsters" in the industry these days, and many projects are doomed from the beginning just because the main goal of such developers is "to try out a new technology" rather than to solve a problem.
I believe that with such a “rusty” logic, the language has a bright future.

## My first project on Rust

I started with building simple CLI.
It is an naive [implementation](https://github.com/sgaliamov/ergo-balance) of a genetic algorithm that looks for an "optimal" balance of keys on a keyboard.

Here is a peace of code from it:

``` rust
pub fn run(population: &mut LettersCollection, context: &Context) -> Result<LettersCollection, ()> {
    let mut mutants: Vec<_> = population
        .into_par_iter()
        .flat_map(|parent| {
            (0..context.children_count)
                .map(|_| parent.mutate(context))
                .collect::<Vec<_>>()
        })
        .collect::<Vec<_>>();

    mutants.append(population);

    let offspring: Vec<_> = mutants
        .into_iter()
        .unique()
        .sorted_by(score_cmp)
        .group_by(|x| x.parent_version.clone())
        .into_iter()
        .map(|(_, group)| group.collect())
        .collect::<Vec<_>>()
        .into_par_iter()
        .flat_map(|group| recombine(group, context))
        .collect::<LettersCollection>()
        .into_iter()
        .unique()
        .sorted_by(score_cmp)
        .into_iter()
        .take(context.population_size)
        .collect();

    if offspring.len() == 0 {
        return Err(());
    }

    Ok(offspring)
}
```

It's the core logic of the application.
It takes some `population` on the input, mutates it, cross the best instances, and returns them.
Simple as piece of cake.

As you can see, it's easy to read and very expressive.
Classic imperative or OOP implementation will require much more code.
And you may notice that it has parallelism on the lines with `into_par_iter`.
It uses [rayon](https://docs.rs/rayon) crate, and it's the great example of the potential of the language.
The idea is very similar with [PLINQ](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq) in `.NET`.

While writhing this code I was really excited, because it feels like you write on high level language like C# or TypeScript.
Functional programming capabilities are impressive for the system level language!

## Comparison with ASP.NET

Since I'm a `.NET` developer I was curious how it performs in comparison with `ASP.NET` platform.
I [expected](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite) to see that the implementation on Rust will over perform C#.
But I have not noticed significant advantage of Rust.
`ASP.NET` even was slightly faster.

To implement `.NET` [version](https://github.com/sgaliamov/rust-web-api/blob/master/dotnet/NotesApi/NotesApi/Controllers/NotesController.cs) I spend less than hour and it's only 55 lines of code including all empty lines.
But to implement Rust version I spend 2 days.

Most probably I did something wrong.

And, most likely, the reason is the notorious I/O operations.
When you create a web server, the programming language is not the most important thing, apparently.
Pure computations on Rust should be [faster](https://benchmarksgame-team.pages.debian.net/benchmarksgame/which-programs-are-fastest.html).

## Pros

While working and reading The Book I collected things that I liked and did not.
Here is my list. Of course, I may miss some important consents and been very subjective, but it's my list after all.

1. Immutability.
1. Performance.
1. Simple, minimalistic language. I love C# but it growing so much last years. I wish Microsoft will do something like JetBrains did with Kotlin. Personally I think we need fresh blood in `.NET`.
1. Takes a lot from FP: If you think that OOP is the only right way to design software, you are very wrong. and if you persist in this, the fate of the dinosaurs awaits you. You will have to maintain old legacy systems, like someone maintain COBOL code now, while rest of the world creates new shiny projects.
   1. shadowing.
   1. strong pattern matching.
   1. immutability. In C# we can define immutability using `readonly` keyword, and we have to define it on design time. with Rust you can decide on the moment whe you use it.
   1. expression returns value from the last line like in `F#`.
   1. `if` can be used as expression `let number = if condition { 5 } else { 6 };` like in `F#`. so no ternary operator.
   1. types inference. not so powerful like in F# or Haskell.
   1. enums are based on `algebraic data types`.
1. Effective iterators, with functionality  like in `C#`.
1. Build in documentation.
1. Build in formatting style.
1. Ownership concept for memory management. Similar with Resource Acquisition Is Initialization (RAII) from C++. No need for GC.
1. Rust is like C++ with best practices embedded in the compiler.
1. Very strict compiler.
1. What makes code safe and enable Zero cost abstraction. Контроль корректности заимствований происходит во время компиляции и не порождает дополнительного исполнимого кода (принцип абстракций с нулевой стоимостью)
1. No `nulls`!
1. No exceptions.
1. Generics.
1. Traits better that interface. Because you can extend any type without changing it. New traits can be implemented for existing types.
1. Build-in test framework. It's interesting idea to have tests mixed with the code. It allows write unit tests much faster. Gives 1re information about intentions. And gives access to internal implementation.
1. Tree shaking?
1. async/.await!
1. messaging for threads.
1. arrow functions with closures.
1. Flexible smart pointers. But it is hard to master them.
1. `Newtype` feature is quite useful to create domain specific types.
1. Easy to read.

## Cons

1. Learning curve is high. You have to read documentation first. Rust actually is a great protection against such developers that managed to learn only one ui framework and thinks they are programmers now. It's too high learning curve for them. So you can be sure that you are in the right company with rustafarians.
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

1. It's not clear when object is moved or copied. `let actual = foo(s);` it depends on whether `s` has `Copy` trait.
1. To get the most interesting things, like async closures, you have to use a nightly build. What is not safe in perspective.

In C# the boxing is just a question on an interview.
This knowledge does not have practical application most of the time.
But in Rust you really use it and if you don't understand it, you will not be able to write code.

Rust will protect your team from such coders that learned one language and one ui framework and think that they are programmers now.



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

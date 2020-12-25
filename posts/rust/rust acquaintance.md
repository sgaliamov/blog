---
created: 2020-10-23
tags: rust, programming languages, reviews, benchmarks, asp.net
---

# My first acquaintance with the Rust language

![Rusted Metal Patterns](https://live.staticflickr.com/4095/4746391656_a4ff32afa6_b.jpg)

["Rusted Metal Patterns"](https://www.flickr.com/photos/99624358@N00/4746391656) by [halseike](https://www.flickr.com/photos/99624358@N00) is licensed under [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/?ref=ccsearch&atype=rich)

## Intro

This post is about my first practical experience of acquaintance with the language.
It can not be an exhaustive overview because I am not a professional `Rust` developer.
And of course I will not tell anything new about the language.
But I think people who are interested in `Rust` should keep the interest and promote it.

To make the post not completely boring, I did a [benchmarking](#comparison-with-aspnet) to compare the implementations based on `r2d2`, `bb8` crates, and `ASP.NET`.

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

## My first Rust project

I started by building a simple CLI.
It is an naive [implementation](https://github.com/sgaliamov/ergo-balance) of a genetic algorithm that looks for an "optimal" balance of keys on a keyboard.

Here is a piece of code from it:

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

This is the core logic of the application.
It takes a certain `population` as the input, mutates it, recombines the best offspring, and returns them for the next iteration.
Piece of cake.
You don't even need to know `Rust` to understand it.

As you can see, it is easy to read and very expressive.
A classic imperative or OOP implementation would require a lot more code.
And you may notice that it has parallelism enabled on the lines with `into_par_iter`.
It uses [rayon](https://docs.rs/rayon) crate, and it's a great example of the language's capabilities and extensibility.
The idea is very similar to [PLINQ](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq) in `.NET`.

When I wrote this code, I was very impressed because it felt like you were writing in a high-level language like `C#` or `TypeScript`.

Functional programming capabilities are very impressive for a system-level language!

## Comparison with ASP.NET

Since I'm mostly a `.NET` developer I was curious how it performs in comparison with `ASP.NET` platform.
I [expected](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite) to see that the Rust implementation outperform C#.




But I have not noticed significant advantage of Rust.
`ASP.NET` even was slightly faster.

TBD table

To implement `.NET` [version](https://github.com/sgaliamov/rust-web-api/blob/master/dotnet/NotesApi/NotesApi/Controllers/NotesController.cs) I spend less than hour and it's only 55 lines of code including all empty lines.
But to implement Rust version I spend 2 days. It's too much, even taking in account that I'm new in this technology stack.

I choose `actix` because it is one of the fastest web servers on Rust.
First I implemented it with `r2d2`. And because `r2d2_postgres` is synchronous I used `actix_web::web::block` function to run queries in a thread pool.

``` rust
#[get("/{id}")]
async fn get_note(
    Path(id): Path<i32>,
    db: web::Data<Pool<PostgresConnectionManager<NoTls>>>,
) -> Result<HttpResponse, Error> {
    let res = web::block::<_, _, r2d2_postgres::postgres::Error>(move || {
        let mut conn = db.get().unwrap();
        let one = conn.query_one(
            "select id, text, timestamp
                from notes
                where id = $1",
            &[&id],
        )?;

        let id: i32 = one.get("id");
        let text: String = one.get("text");
        let timestamp: DateTime<Utc> = one.get("timestamp");

        Ok(Note {
            id,
            text,
            timestamp,
        })
    })
    .await
    .map(|x| HttpResponse::Ok().json(x))
    .map_err(|_| HttpResponse::InternalServerError())?;

    Ok(res)
}
```

Nothing is criminal in this code.
Yes, it's verbose, but it's clear that it has no over-engineering and overhead.
So it should be fast.
I thought.

But it was slower that ASP.NET.
It was a surprise for me.
I thought that the reason is because I do not have normal asynchronicity.
Probably I did something wrong.
I will be happy if someone tell me how to improve it.

So I decided to try `bb8`. The code that I've got at the end is even more [verbose](https://github.com/sgaliamov/rust-web-api/blob/master/src/bin/bb8.rs). So much, that I even shame to show it here. Except of complexity and fights with the compiler, I faced another not nice thing about Rust. To use some "obvious" features, like asynchronous closures, you have to use nightly build to enable [unstable features](https://doc.rust-lang.org/stable/unstable-book/the-unstable-book.html).

Most likely, the reason is the notorious I/O operations.
Or it's because of immature libraries or because of nightly build is slower that stable.
I don't know.
The fact is, regular developer like me will not gain performance boost from using Rust when creates a web api server.

When you create a web api server, the programming language is not the most important thing, apparently.

Pure computations on Rust should be [faster](https://benchmarksgame-team.pages.debian.net/benchmarksgame/which-programs-are-fastest.html).

Even so, I really enjoy to write Rust code. It's a really stir mind and

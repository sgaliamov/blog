---
title: My first acquaintance with the Rust language
categories: ["rust", "review", "benchmark", "asp.net"]
created: 2020-10-23
date: 2020-12-27
layout: post
---

This post is about my first practical experience of acquaintance with the language.

<p style="font-size: 0.9rem;font-style: italic;"><img style="display: block;height: 256px" src="https://live.staticflickr.com/7511/16131487998_3b484d34af_b.jpg" alt="Rusty Contraption"><a href="https://www.flickr.com/photos/99649389@N02/16131487998">"Rusty Contraption"</a><span> by <a href="https://www.flickr.com/photos/99649389@N02">darkday.</a></span> is licensed under <a href="https://creativecommons.org/licenses/by/2.0/?ref=ccsearch&atype=html" style="margin-right: 5px;">CC BY 2.0</a><a href="https://creativecommons.org/licenses/by/2.0/?ref=ccsearch&atype=html" target="_blank" rel="noopener noreferrer" style="display: inline-block;white-space: none;margin-top: 2px;margin-left: 3px;height: 22px !important;"><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc_icon.svg?image_id=7c061de4-f173-4908-81dd-dd1663a5aa47" /><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc-by_icon.svg"/></a></p>

It can not be an exhaustive overview because I am not a professional `Rust` developer.
And of course I will not tell anything new about the language.
But I think people who are interested in `Rust` should keep the interest and promote it.

To make the post not completely boring, I did a [benchmarking](#comparison-with-aspnet) to compare the implementations based on `r2d2`, `bb8` crates, and `ASP.NET`.

So, after ten years of development and five years after the [official release](https://blog.rust-lang.org/2015/05/15/Rust-1.0.html), `Rust` should be mature enough.
Right?
When it has been ranked as the ["The Most Loved Programming Language"](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-languages-loved) for five years in a row(!), you should definitely try it.
And I've tried.
I've created two simple projects to learn it.

In my research, I discovered some facts about `Rust` that I find interesting.

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
It takes a certain population as the input, mutates it, recombines the best offspring, and returns them for the next iteration.
Piece of cake.
You don't even need to know `Rust` to understand it!

As you can see, it is easy to read and very expressive.
A classic imperative or OOP implementation would require a lot more code.
And you may notice that it has a parallelism enabled on the lines with `into_par_iter`.
It uses [rayon](https://docs.rs/rayon) crate, and this is a great example of the language's capabilities and extensibility.
The idea is very similar to `AsParallel` from [PLINQ](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq) in `.NET`.

When I wrote this code, I was very impressed because it felt like you were writing in a high-level language like `C#` or `TypeScript`.

Functional programming capabilities are very impressive for a system-level language!

## Comparison with ASP.NET

Since I'm mostly a `.NET` developer I was curious how it performs in comparison with `ASP.NET` platform.
I [expected](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite) to see that the Rust implementation will outperform C#.

For this I have implemented a simple [web server](https://github.com/sgaliamov/rust-web-api) that can add and return notes from `PostgreSQL`.
I also added an endpoint that returns the current date to have some logic without I/O.
I chose [actix](https://actix.rs/) because it is one of the fastest web frameworks for Rust.

It took me less than an hour to implement the `.NET` [version](https://github.com/sgaliamov/rust-web-api/blob/master/asp.net/NotesApi/Controllers/NotesController.cs), which is just 55 lines of code, including all blank lines and line breaks, but it took me more than 2 days to implement the Rust version.
This is too much even considering that I am new to this technology stack.
And the code that I ended up with to is at least twice as verbose.

My implementation was originally based on [r2d2](https://github.com/sfackler/r2d2). It's a generic connection pool for Rust.
And since `r2d2_postgres` does not support asynchronous execution, I used the `actix_web::web::block` function to execute requests on the thread pool.

Here is the implementation of the `get_note` method:

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

There is nothing criminal in this code.
Yes, it's verbose, but it's clear that there are no over-engineering and overhead costs.
So it should be fast.
I thought.

Always profile before optimizing!

I created a simple benchmarking [project](https://github.com/sgaliamov/rust-web-api/blob/master/asp.net/Benchmark/Program.cs) using [BenchmarkDotNet](https://benchmarkdotnet.org/articles/overview.html).
And it showed that `ASP.NET` is 60% faster!

### ASP.NET benchmark

| Parallel | Path       |         Mean |       Error |      StdDev |       Median |
| -------- | ---------- | -----------: | ----------: | ----------: | -----------: |
| True     | GET /date  |     9.764 ms |   0.1692 ms |   0.1500 ms |     9.713 ms |
| False    | GET /date  |    43.043 ms |   0.8405 ms |   0.7862 ms |    42.742 ms |
| True     | POST & GET |   589.727 ms |  40.3020 ms | 118.8312 ms |   596.048 ms |
| False    | POST & GET | 3,157.214 ms | 231.7176 ms | 675.9310 ms | 3,165.454 ms |

### R2D2 benchmark

| Parallel | Path       |         Mean |       Error |      StdDev |       Median |
| -------- | ---------- | -----------: | ----------: | ----------: | -----------: |
| True     | GET /date  |     8.945 ms |   0.2274 ms |   0.6596 ms |     8.935 ms |
| False    | GET /date  |    39.317 ms |   0.7753 ms |   1.7969 ms |    38.967 ms |
| True     | POST & GET |   976.429 ms |  51.5924 ms | 152.1214 ms |   966.551 ms |
| False    | POST & GET | 5,042.993 ms | 281.0737 ms | 828.7520 ms | 5,045.977 ms |

Yes, the end point of getting the date is slightly faster on `actix`, but the difference is negligible.

It was a surprise for me.
I thought the reason was because I don't have normal async.

So I decided to try [bb8](https://docs.rs/bb8/0.6.2/bb8/).
It's similar to `r2d2`, but designed for asynchronous connections.
The code that I've got at the end is even more [verbose](https://github.com/sgaliamov/rust-web-api/blob/master/src/bin/bb8.rs).
Apart from the complexity and fights with the compiler, I ran into another annoying Rust quirk.
In order to use some "obvious" features, like asynchronous closures, you have to use nightly build to enable [unstable features](https://doc.rust-lang.org/stable/unstable-book/the-unstable-book.html).

But even `bb8` is cuter, it is only slightly better than `r2d2`. ~~Empire~~ Microsoft is still winning.

### BB8 benchmark

| Parallel | Path       |         Mean |       Error |       StdDev |       Median |
| -------- | ---------- | -----------: | ----------: | -----------: | -----------: |
| True     | GET /date  |     9.054 ms |   0.2258 ms |    0.6478 ms |     8.990 ms |
| False    | GET /date  |    37.105 ms |   0.7373 ms |    1.6941 ms |    36.482 ms |
| True     | POST & GET |   888.076 ms |  48.6239 ms |   142.605 ms |   867.573 ms |
| False    | POST & GET | 5,031.185 ms | 368.6956 ms | 1,087.107 ms | 4,895.177 ms |

Most likely, the reason is the notorious I/O operations.
Either it's because of immature libraries, or because nightly builds are slower than stable builds.
I don't know.
The fact remains.
An average developer like me won't get the performance gain from using Rust when building a web api server.
Probably I did something wrong.
I will be happy if someone tell me how to improve it.

When you create a web api server, the programming language is not the most important thing, apparently.
However, pure computations on Rust should be [faster](https://benchmarksgame-team.pages.debian.net/benchmarksgame/which-programs-are-fastest.html).

No matter what, I really enjoyed writing code in Rust.
I will continue to use it if I need to write efficient calculations or garbage collection will be a problem.

---
title: Evaluating keyboard layouts for ErgoDox EZ
categories: ["ErgoDox", "reviews"]
created: 2020-08-09
date: 2021-02-07
layout: post
---

When you buy a keyboard that costs almost 400€, you probably ~~insane~~ want to use it 100%.
Initially I planned to configure the peripheral buttons only.
But since with a non-standard keyboard you have to learn how to type again, I thought "why not learn a new layout?"

The only [Dvorak](https://en.wikipedia.org/wiki/Dvorak_keyboard_layout) came to mind.
After a short investigation it's turned out that Dvorak is not much better than Qwerty.
And some people aren't particularly keen on [Colemak](https://colemak.com/).

For that reason people have created quite a few different layouts.
In addition to well known Dvorak and Colemak we have:

- [Workman](https://workmanlayout.org/).
- [Carpalx](http://mkweb.bcgsc.ca/carpalx/).
- [Capewell](http://www.michaelcapewell.com/projects/keyboard/layout_capewell.htm).
- [Norman](https://normanlayout.info/).
- and many others. In my research I've found around 25 different designs.

As it usual happens, a great variety slows down and [complicates](https://en.wikipedia.org/wiki/Overchoice) a selection process.
To make an informed choice, I decided to create a tool that will help me with this.

## Why create another tool?

There are many tools and methodologies out there already that evaluate keyboard layouts.
But they all have the one [fatal flaw](https://www.drdobbs.com/windows/a-brief-history-of-windows-programming-r/225701475): they are based on a standard keyboard.

Most standard keyboards are shifted: upper rows slightly moved left relatively lower rows.
What is completely reasonable for the right hand, but not for the left.
The only explanations, that I can find for why do we have the same shift for the left hand, is reactionism, and is that manufacturers want to save money on production.

ErgoDox keyboard is very different.
Except for the extra thumb keys, it's slightly bigger than a typical laptop keyboard, and all buttons are placed strictly vertical.
On a regular keyboard for example, you don't have to tilt your wrist when you press the button N because of the shifting.
On the other hand, in the literal and figurative sense, the button B is more reachable now on ErgoDox.

On a regular keyboard it's much easier to use pinky fingers on the top row.
On my laptop I can reach P and Q without any efforts, but on ErgoDox it's not really comfortable.
Maybe this is because of "bad habits" or the adaptation period, but I just don't see how should I place my hands to fully enable pinkies on ErgoDox.

## Estimation method

What I find important:

1. <ins>Support for modern English only</ins>

    For this, I used technical literature containing about 40 MB of pure text to make comparisons.
    [Statistics](https://github.com/sgaliamov/ergo-layouts/blob/master/docs/statistics.md) that I've collected is slightly differs from that you can get using "Alice's Adventures in Wonderland" or "Moby Dick or The Whale".
    This is fine for me, because I deal with technical texts more often.

1. <ins>Alternate hands less</ins>

    Colemak is often criticized for the fact that T and H keys are placed on the different sides of a keyboard.
    It breaks the rhythm of the typing.
    This and the fact that the one of the most used vowels, A and O, are intended for pinkies, turned me away from using Colemak.
    For the same reason it's not wise to place all vowels on the same side, like Dvorak does.

1. The keys of the <ins>most commonly used [digraphs](https://en.wikipedia.org/wiki/Digraph_(orthography))</ins> should be placed close to each other.

    For example, E and R letters create the most used combination in English.
    Even Qw**er**ty has it, but many popular layouts (Workman, Norman, Colemak, etc.) ignore this for a reason that I can not explain.

1. <ins>Type continuously with one finger less</ins>

    It is not enough to simply place keys for digraphs close to each other.
    It's slower to type when paired letters are placed vertically, because a finger need to travel some distance to reach a next position.
    Much comfortable and faster to press the next key with another finger.
    For that reason, it's a big loss for Workman and Dvorak to put O and E in the same row next to each other, this combination is used very rarely.
    So, much wiser to have them separately to be able to place consonants in between.

1. <ins>Minimize usage of pinkies</ins>

    The little finger is already overused when pressing `Shift` and `Ctrl` buttons. For the weakest finger it's too big load.

1. <ins>The usage of hands should be balanced</ins>

    It's hard to say which balance is the best.
    It sounds logical that the load on hands should be equal.
    But from other point of view, the left hand is used more often because we use a mouse with the right hand and place the most of shortcuts on the left side.
    To not over-complicate, I decide stick to balanced option.

1. <ins>Punctuation marks do not have to be in the standard positions</ins>

    Since they are completely legitimate elements of a text, we can benefit from placing them on a better positions.
    Dot is used more often than a half of letters.
    Why keep it on an awkward button?

1. <ins>Shortcuts are not really important</ins>

    Since we use ErgoDox we can configure extra layers and have almost any shortcut anywhere.
    The authors of many layouts deliberately left the Z, X, C, V buttons in place to preserve the familiar experience.
    It makes sense for regular keyboards, and I would care about it if I didn't have ErgoDox.\
    I thought initially.\
    The biggest problem with that is when you have to switch to another language, all shortcuts are moved to the original places and you have to remember two sets of shortcuts.
    This is really annoying.
    But if you type in one language only this is still relevant to you.

1. <ins>Estimation is based on "effort"</ins> that you have to apply when you press a key <ins>and a distance</ins> between two taps. This distance factor helps keep most commonly used pairs close to each other.

Having all this requirements I started [this](https://github.com/sgaliamov/ergo-layouts) project.
In addition, I planned to try F#, and scope of this challenge was perfect for this.
Why do I think that this estimation method is applicable?
Because the results that it gives correlates with other estimations.
Qwerty is bad, MTGAP is good as expected.

ErgoDox is such a keyboard for which you first need to learn a functional programming language with [static typing](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system), and then create a utility to find the most optimal configuration for you.
Joke.
A functional language with [dynamic typing](https://en.wikipedia.org/wiki/Lisp) is also suitable.

## Results

Here is the summary tables:

| Layout          | Result      | Hand switch | Efforts     | Distance    |
| --------------- | ----------- | ----------- | ----------- | ----------- |
| Capewell 9.2    | 45193617.65 | 47.041394   | 98832840.65 | 47665034.04 |
| Norman          | 45525407.6  | 50.455266   | 99088143.5  | 46649178.52 |
| Mtgap V2        | 45883461.92 | 48.539812   | 98392443.3  | 48091639.02 |
| Workman         | 47445482.48 | 53.213322   | 104341732.7 | 49446477.14 |
| Arensito        | 47806418.04 | 50.737255   | 108640305.3 | 47044747.91 |
| C-Qwerty        | 47938881.83 | 48.290954   | 107675320.2 | 48878498.87 |
| Collemak        | 47942703.73 | 51.910584   | 107475042.2 | 49575367.82 |
| Capewell 9.3    | 48011209.23 | 47.032259   | 110323403.1 | 49490581.12 |
| C-Qwerty N      | 48470886.62 | 48.290954   | 108304816.1 | 47669391.21 |
| QWERF           | 48516926.96 | 48.290954   | 107871985.1 | 50087717.89 |
| Niro            | 48696790.15 | 49.399818   | 108290860.6 | 48582745.21 |
| Soul            | 49135335.44 | 54.686361   | 107494308   | 49559092.46 |
| Breakl15        | 49162860.35 | 65.826767   | 112285052.4 | 42820649.45 |
| Asset           | 49421422.83 | 50.501631   | 109737945.1 | 47750225.2  |
| C-Qwerty 2      | 49594136.64 | 48.290954   | 112471260.7 | 52014351.36 |
| Capewell-Dvorak | 51115005.05 | 61.646022   | 120920054.6 | 45705076.4  |
| Qfmlwy          | 51182598.68 | 65.240855   | 121032878.6 | 43556445.46 |
| Qwerty          | 51443513.18 | 48.290954   | 114245001.9 | 54112473.19 |
| Qgmlwy          | 51730501    | 63.99991    | 124924518.5 | 44631754.31 |
| Mtgap V1        | 51763536.69 | 61.640297   | 122502251.4 | 43485922.36 |
| Kaehi           | 51846819.67 | 64.727373   | 118732968.5 | 45017963.69 |
| Qgmlwb          | 51912589.82 | 64.228367   | 125083505.1 | 44638433.83 |
| Gelatin         | 52398205.69 | 61.918502   | 121025427.9 | 45537721.82 |
| Klausler        | 53374659.21 | 63.307755   | 129126626.8 | 46647430.92 |
| Dvorak          | 54172386.37 | 65.015753   | 134566516.9 | 45259664.53 |
| Tnwmlc          | 57007114.11 | 40.363025   | 125989034.5 | 61155802.13 |

| Layout          | Left hand | Right hand | Left hand continuos | Right hand continuos |
| --------------- | --------- | ---------- | ------------------- | -------------------- |
| Capewell 9.2    | 52.08266  | 47.91734   | 24.04615            | 20.592579            |
| Norman          | 52.456099 | 47.543901  | 24.800316           | 17.330407            |
| Mtgap V2        | 47.084201 | 52.915799  | 20.097583           | 22.864456            |
| Workman         | 49.522372 | 50.477628  | 20.53113            | 18.425939            |
| Arensito        | 48.408069 | 51.591931  | 18.491889           | 22.58691             |
| C-Qwerty        | 58.197649 | 41.802351  | 31.516131           | 13.038066            |
| Collemak        | 48.393806 | 51.606194  | 19.979273           | 20.086835            |
| Capewell 9.3    | 52.277813 | 47.722187  | 23.89154            | 20.752456            |
| C-Qwerty N      | 58.197649 | 41.802351  | 31.516131           | 13.038066            |
| QWERF           | 58.197649 | 41.802351  | 31.516131           | 13.038066            |
| Niro            | 55.464785 | 44.535215  | 28.199457           | 14.990183            |
| Soul            | 49.371712 | 50.628288  | 19.102899           | 18.335315            |
| Breakl15        | 46.233075 | 53.766925  | 8.945973            | 17.240867            |
| Asset           | 52.164623 | 47.835377  | 24.475287           | 17.564965            |
| C-Qwerty 2      | 58.197649 | 41.802351  | 31.516131           | 13.038066            |
| Capewell-Dvorak | 49.861888 | 50.138112  | 14.715419           | 15.674638            |
| Qfmlwy          | 50.216918 | 49.783082  | 14.682388           | 12.445085            |
| Qwerty          | 58.197649 | 41.802351  | 31.516131           | 13.038066            |
| Qgmlwy          | 48.382126 | 51.617874  | 13.541359           | 14.696425            |
| Mtgap V1        | 51.182863 | 48.817137  | 16.019137           | 14.298194            |
| Kaehi           | 49.54177  | 50.45823   | 14.350148           | 13.249166            |
| Qgmlwb          | 49.349946 | 50.650054  | 14.326982           | 13.790358            |
| Gelatin         | 47.974273 | 52.025727  | 14.373887           | 15.986106            |
| Klausler        | 48.336339 | 51.663661  | 12.266107           | 16.381533            |
| Dvorak          | 45.45322  | 54.54678   | 8.871743            | 18.2422              |
| Tnwmlc          | 71.6472   | 28.3528    | 48.633145           | 4.436925             |

| Layout          | Same finger | Outward rolls | Inward rolls |
| --------------- | ----------- | ------------- | ------------ |
| Capewell 9.2    | 2.00065     | 1.325089      | 1.121522     |
| Norman          | 4.702983    | 2.673737      | 4.182726     |
| Mtgap V2        | 2.219063    | 1.216561      | 1.604332     |
| Workman         | 2.367174    | 1.846702      | 1.687218     |
| Arensito        | 1.569142    | 1.049794      | 1.263978     |
| C-Qwerty        | 5.345756    | 3.181462      | 3.901224     |
| Collemak        | 4.5349      | 0.813623      | 1.73332      |
| Capewell 9.3    | 2.066182    | 1.394741      | 1.139836     |
| C-Qwerty N      | 4.922597    | 2.966113      | 3.736252     |
| QWERF           | 4.72257     | 3.054381      | 3.944513     |
| Niro            | 3.495869    | 1.603085      | 2.351795     |
| Soul            | 2.383072    | 1.152669      | 1.562227     |
| Breakl15        | 2.436063    | 1.942188      | 1.793562     |
| Asset           | 4.281506    | 1.787859      | 2.452479     |
| C-Qwerty 2      | 5.364739    | 2.941701      | 3.462195     |
| Capewell-Dvorak | 3.327597    | 1.819766      | 2.436567     |
| Qfmlwy          | 3.804616    | 2.019231      | 2.200393     |
| Qwerty          | 5.581466    | 2.97415       | 3.432625     |
| Qgmlwy          | 4.028855    | 2.057566      | 2.548201     |
| Mtgap V1        | 1.466548    | 1.0598        | 0.963373     |
| Kaehi           | 2.348925    | 1.318246      | 1.706143     |
| Qgmlwb          | 3.822031    | 2.225356      | 2.233585     |
| Gelatin         | 3.344234    | 1.893624      | 2.205023     |
| Klausler        | 2.634559    | 1.040643      | 1.24794      |
| Dvorak          | 2.523221    | 1.234723      | 1.922681     |
| Tnwmlc          | 15.660386   | 7.58605       | 7.305419     |

Columns description:

1. **Result** - the accumulative score for the layout.
1. **Hand switch** - how often you have to alternate hands during a typing.
1. **Efforts** - summary effort of tapping keys.
1. **Distance** - summary distance between keys during a typing.
1. **Same finger** - how often you have to use the same finger.
1. **Left hand** - how often you have to use fingers on the left hand.
1. **Right hand** - how often you have to use fingers on the right hand.
1. **Left hand continuos** - how long you use the left hand without switching to the right.
1. **Right hand continuos** - how long you use the right hand without switching to the left.
1. **Outward rolls** - how often you fingers moves up.
1. **Inward rolls** - how often you fingers moves a lower row.

Links to all complementary files:

1. [All layouts results](https://github.com/sgaliamov/ergo-layouts/blob/master/docs/layouts.md).
1. [Summary table](https://github.com/sgaliamov/ergo-layouts/blob/master/docs/results.xlsx).
1. [Configuring Excel file](https://github.com/sgaliamov/ergo-layouts/blob/master/docs/layouts.xlsx) that was used to do evaluations and the design.
1. [Statistics](https://github.com/sgaliamov/ergo-layouts/blob/master/docs/statistics.md) of sampling texts.

## Own layout

In the process of creating the evaluative method it became clear that none of the existing keyboards would fit it.
And even the first timid attempt to create a layout gave good results.
I was inspired and spent a lot of time trying different options, but I could not be satisfied with the results.

I realized one important thing: **there is no perfect layout for keyboard**.
Everything very depends on the estimation method, used keyboard, sampling texts, and personal preferences of an author.
It will always be a compromise.
It all depends on what you think is important.
Almost every attempt based on statistics and common sense will provide better result than Qwerty and Dvorak.
The time, that you spend on configuring it and learning how to type, most probably will not pay off.

I could not drop this  idea and thus I started another [project](https://github.com/sgaliamov/ergo-balance) in which, using a genetic algorithm, I tried to find the optimal layout.
This time I decided to use Rust.
It was an interesting experience.
But more on that another time.

## Summary

So, as you can see, that Dvorak, which is the most popular alternative to Qwerty, is actually the worst you can choose.
Tnwmlc is not counted because it was artificially designed to be [bad](http://mkweb.bcgsc.ca/carpalx/?worst_layout).

If you need a good layout I recommend to look at:

1. [Capewell](http://www.michaelcapewell.com/projects/keyboard/layout_capewell.htm) is good balanced keyboards that are produced using [genetic algorithm](https://en.wikipedia.org/wiki/Genetic_algorithm).
1. [Colemak](https://colemak.com/) not really bad if you switch D and H keys. It's much better alternative than Dvorak. And is quite popular and has good support.
1. If you are not happy with Colemak you can look at [Norman](https://normanlayout.info/) or [Workman](https://workmanlayout.org/). Very worthy options.

## Useful links

- <http://patorjk.com/keyboard-layout-analyzer/#/main>.
- <https://elliotgeorge.net/2018/11/22/the-kaehi-keyboard-layout/>.
- <http://mkweb.bcgsc.ca/carpalx/>.
- <https://kennetchaz.github.io/symmetric-typing/results.html>.
- <https://geekhack.org/index.php?topic=67604.50s>.
- <https://mtgap.wordpress.com/2009/08/07/optimized-evolutionary-algorithm-for-keyboard-design-part-1/>.

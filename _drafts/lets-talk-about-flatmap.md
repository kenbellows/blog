---
title:  "Let's talk about `flatMap`"
date: Thu, Feb 08, 2018 12:34:36 PM
layout: post
categories: javascript, coding, esnext
---

# Let's talk about `flatMap()`

I learned something this morning via [this tweet](https://twitter.com/jaffathecake/status/961585899275542528) from
Jake Archibald ([@jaffathecake](https://twitter.com/jaffathecake)) that is deeply exciting to me: JavaScript Arrays
are getting `.flatten()` and, most exciting to me, `.flatMap()`!

<blockquote class="twitter-tweet" data-lang="en">
  <p lang="en" dir="ltr">
    arr.flatMap() and arr.flatten() coming to WebKit <a href="https://t.co/JrMhOursao">https://t.co/JrMhOursao</a>
  </p>&mdash; Jake Archibald (@jaffathecake) 
  <a href="https://twitter.com/jaffathecake/status/961585899275542528?ref_src=twsrc%5Etfw">February 8, 2018</a>
</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I enthusiastically passed along the news to a coworker of mine who's a more recent developer, previously a fully self-
taught PHP developer before getting into web development stuff. This coworker is not the sort of person who keeps up
with all the fanciest trends and patterns and such in the development world (*engage snooty scoffing*), so he wasn't
familiar with `flatMap()` or `flatten()` as concepts, and was... undewhelmed, I guess, when I explained it. His
response was sort of a "huh, okay, and why do I care about that again? Seems interesting, but when would I want it?"

So I thought, fair enough, I can definitely understand that perspective. `flatMap()` in particular is one of those
functions that seems pretty trivial and boring at first glance, but when you get into it, it turns out to be super
powerful and impressive, and it enables really elegant solutions to some previously messy problems.

I thought about it, Googled around a bit for good example use cases, and came up with one of my own that I thought
would appeal to my coworker, and I thought I'd share it here to add it to the list.

But first let's talk about what `flatMap()` actually does.

## What `flatMap()` Is

To borrow from [2ality's great post about `flatMap()` and `flatten()`](http://2ality.com/2017/04/flatmap.html), the
easiest way to explain `flatMap()` is probably by comparison with `map()` and `reduce()`. In fact, an easy way to
implement your own `flatMap()` polyfill is with a chained `map()` and `reduce()`, but I'm getting ahead of myself.

Here's the main point: when you call `someArray.map(mapFn)`, every call to `mapFn` must return a single value, and
the output array will always, by definition, have the same length as the input array. A dumb example:

    [1,2,3,4].map(n => n*2)  //-> [2, 4, 6, 8]

If you ever want to produce multiple output values from a single input, you can return an array of values, but the
return value from your `.map()` call will be an array of arrays. Which is cool, if that's what you're into, and
especially if you need to keep the outputs from each input grouped separately:

    [1,2,3,4].map(n => n%2 ? n : [n/2, n])  //-> [1, [1, 2], 3, [2, 4]]

But sometimes you don't and you just want a single big, *flat* array with all the outputs, regardless of their
input. And as you may have guessed, this is where `.flatMap()` comes in.

By contrast with `.map()`, when you call `someArray.flatMap(mapFn)`, each call to `mapFn` can return a single value
or an array of values; when an array is returned, it will be concatenated onto the resulting array so that its
*elements* are added to the results, rather than the array *itself* becoming an element in the array:

    [1,2,3,4].flatMap(n => n%2 ? n : [n/2, n])  //-> [1, 1, 2, 3, 2, 4]

A cool side-effect of this behavior is that you can implicitly filter out values whose output you don't want by
simply returning an empty array:

    [1,2,3,4].flatMap(n => n%2 ? [] : [n/2, n])  //-> [1, 2, 2, 4]

So to sum up the difference between `.map()` and `.flatMap()` in a single sentence, and to paraphrase the 2ality
article I linked above, `.map()` translates each element of the input array to exactly one output value, but
`.flatMap()` can translate each element of the input array to zero or more output values.

Cool, but... why would you want that?


## My example

Here's just one use case I dreamed up in a couple minutes to explain the value of `.flatMap()` to my coworker. It's
about splitting spritesheets and rejoining them into larger spritesheets. I don't know how realistic this example
actually is, as I don't have a ton of practical experience myself in the area of spritesheets, but I think it should
be enough to get the point across and start the brain juices flowing.

In his free time, my coworker is a game developer. He has written and continues to develop a web-based RPG with
3D mobs and such. One of the things he deals with quite often, and something we've chatted about a lot as he's worked
through it, is spritesheets. We've talked about the pros and cons of different sizes of spritesheet files, whether to
produce a separate spritesheet per mob, or one larger sheet per map area, or one for the whole game, how all this
affects performance, compression, resolution, etc... all that good stuff. So I thought up a `.flatMap()` example that
I figured he would relate to.

Imagine you're developing a sprite-heavy game like my coworker's. Suppose you've decided that the best way to handle
your sprites is to package up several sprites in one sheet. You're using a library that will pack the frames of your
various sprites into a single image file in the most efficient way possible, and generate the necessary data file so
that your client-side rendering library knows how to find the frames for each mob, wherever they end up in the sheet.

Se let's assume that this library (we'll call it SpriteFoo for the sake of having a name) exposes these two functions:

  - `SpriteFoo.getFrames({img: Image, frameWidth: Number, frameHeight: Number}):Image[]` - accepts a spritesheet
    Image and the pixel dimensions of each frame in that image, loads the image, splits it into individual frames, and
    returns the frames as an array of Image objects

  - `SpriteFoo.makeSpritesheet(Image[]):Image` - accepts an array of Image objects containing sprite frames,
    potentially of completely different dimensions, algorithmically determines an efficient way to pack them into a
    single sheet to produce the smallest possible file size, and returns the final packed spritesheet Image

Now suppose you have an array of spritesheet Image objects representing all the mobs in a given map area, and you want
to pack them into a single spritesheet for that area. You'll need to call `SpriteFoo.getFrames()` for each
spritesheet, which will return an array of Image objects containing the frames for that sprite, then you'll need to
combine all those Image arrays into a single array to pass to `SpriteFoo.makeSpritesheet()`.

Without using `.flatMap()`, you'd need to do something like this:

  function combineSpritesheets(spritesheets) {
    let allFrames = []
    spritesheets.forEach(sheet => {
      allFrames = allFrames.concat(SpriteFoo.getFrames(sheet))
    })
    return SpriteFoo.makeSpritesheet(allFrames)
  }

That's *okay*, but it's a little less functional than it could be, and it feels a little bit brittle. You could use
`.reduce()` to clean things up a little:

  function combineSpritesheets(spritesheets) {
    return SpriteFoo.makeSpritesheet(
      spritesheets
        .map(SpriteFoo.getFrames)
        .reduce((all, frames) => all.concat(frames), []) 
    )
  }


That's definitely nicer, but it's still a bit verbose for my taste. Now, once you have `.flatMap()`, it all
becomes much cleaner:

  function combineSpritesheets(spritesheets) {
    return SpriteFoo.makeSpritesheet(spritesheets.flatMap(SpriteFoo.getFrames))
  }

That's it! One line! How elegant is that? Each call to `SpriteFoo.getFrames()` returns an array of Image objects, which
`.flatMap()` cleanly adds to that single output array. Once all sheets have been split, a single array with all the
frames is passed right to `SpriteFoo.makeSpritesheet()`.




## Further reading

- [Jake Archibald's Tweet](https://twitter.com/jaffathecake/status/961585899275542528)
- [The WebKit changeset he linked that implements `Array.prototype.flatten()` and `Array.prototype.flatMap()`](https://trac.webkit.org/changeset/228266/webkit)
- ["Functional pattern: flatMap" - 2ality](http://2ality.com/2017/04/flatmap.html)



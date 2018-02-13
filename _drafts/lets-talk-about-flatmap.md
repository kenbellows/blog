---
title:  "Let's talk about `flatMap`"
date: Tue, Feb 13, 2018 09:02:34 AM
layout: post
categories: javascript, coding, esnext
---


# Let's talk about `.flatMap()`

I learned something recently via [this
tweet](https://twitter.com/jaffathecake/status/961585899275542528) from Jake Archibald
([@jaffathecake](https://twitter.com/jaffathecake)) that is deeply exciting to me: JavaScript
Arrays are getting `.flatten()` and, most exciting to me, `.flatMap()`!

<blockquote class="twitter-tweet" data-lang="en">
  <p lang="en" dir="ltr">
    arr.flatMap() and arr.flatten() coming to WebKit
    <a href="https://t.co/JrMhOursao">https://t.co/JrMhOursao</a>
  </p>
  &mdash; Jake Archibald (@jaffathecake)
  <a href="https://twitter.com/jaffathecake/status/961585899275542528?ref_src=twsrc%5Etfw">
    February 8, 2018
  </a>
</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

In case I haven't said it enough times yet, I'm *pumped*. These may seem like trivial
additions, but they're important, `.flatMap()` in particular. There's seriously an entire
sub-field of mathematics and computer science research, the study of monads within the broader
field of category theory, that basically boils down (with *much* oversimplification) to
describing all the neat things you can do with `.flatMap()`.

I know there's a certain feeling in a not insignificant part of the dev community that adding
more and more stuff to a language, as has been happening a *lot* lately to JavaScript, HTML,
and CSS, can be a bad thing, and that it's better to stabilize on a core set of functionality
that everyone knows, and focus on discovering and refining best practices and patterns within
that shared knowledge base rather than adding a bunch of new stuff to the language every other
week. I can understand that perspective; I've felt the undertow of the rapid web framework
churn over the last couple years, too (though it seems to have settled down a lot in 2017). But
I have to be real: I *love* all the stuff coming to web languages. ESNext fills my dreams with
happiness. If I may play my old-school-geek-cred card, I've been writing JavaScript since I was
a stereotypical socially inept 12-year-old messing around with Notepad and Internet Explorer on
a crappy Windows 98 box my IT Guy&trade; dad reclaimed from his company's trash heap, and I
have never been more excited about the direction the language is going. Features like
`.flatMap()` are not just flashy extras that nobody needs; they're usually legitimately really
useful stuff that will makes our lives as web developers so much better.

Anyway, enough of the proselytizing.

My point here is actually to cater a bit to the folks who question how useful a function like
`.flatMap()` really is. There are a hundred articles already out there explaining how it works,
why it's the most elegant thing since sliced arrow functions, all that jazz, and I'm sure this
news will produce a thousand more in the next while. But in my experience, these articles tend
to go heavy on rhetoric and theory and toy examples, and light on practical demo code that
gives a sense of how you might actually use it to improve your life. So that's what I want to
do: lay out some down-to-earth examples that will not merely *tell* but *show* you how helpful
`.flatMap()` can be.

Buuuut... before we do that, for those not up to speed, let's do the thing I just criticized
and talk for a quick minute about what `.flatMap()` is and show a trivial toy example.


## What `flatMap()` Is

Up front, I'll say that if you want a much better and more thorough rundown on `.flatMap()` and
how it works, please see [2ality's awesome post over here about `flatMap()` and
`flatten()`](http://2ality.com/2017/04/flatmap.html). This section will basically be a summary
and a paraphrase of some of that article, with my own examples.

The obvious easiest way to explain `flatMap()` is by comparison with `map()`. The main point
is that when you call `someArray.map(mapFn)`, every call to `mapFn` must return a single value;
the output array will always, by definition, have the same length as the input array:

```javascript
[-1,0,1,2,3,4].map(n => n*2)  //-> [-2, 0, 2, 4, 6, 8]
```

This is often what you want, but there are two cases this doesn't cover: you may want to map a
single value to *multiple* values in your output, or you may want to map certain values to
*zero* outputs, effectively filtering out certain inputs. These can of course both be achieved
with chained calls to `.filter()` or `.reduce()`, that sort of thing, but wouldn't it be nice
if you could elegantly wrap all of that into a single, tight, efficient, readable function?

Using only `.map()`, if you ever want to produce multiple or zero output values from an input,
you can of course return an array of values for each input and deal with that later:

```javascript
[-1,0,1,2,3,4].map(n => n < 0 ? [] : n%2 ? n : [n/2, n])  //-> [[], [0, 0], 1, [1, 2], 3, [2, 4]]
```

But sometimes you just want a single, big, *flat* array with all the outputs at the same level,
no nesting, regardless of the number of inputs. And of course, this is where `.flatMap()`
comes in.

As with `.map()`, when you call `someArray.flatMap(mapFn)`, each call to `mapFn` can return a
single value or an array of values, but in constrast, whenever an array is returned, it will
be *concatenated* to the resulting array, not just appended, so that its *elements* are added
to the results, rather than the array *itself* becoming an element in the array:

```javascript
[-1,0,1,2,3,4].flatMap(n => n < 0 ? [] : n%2 ? n : [n/2, n])  //-> [0, 0, 1, 1, 2, 3, 2, 4]
```

So to sum up the difference between `.map()` and `.flatMap()` in a single sentence, and to
again paraphrase the 2ality article I linked above:

> `.map()` translates each element of the input array to exactly one output value, but
> `.flatMap()` can translate each element of the input array to **zero or more** output values.


## The Examples

Okay, now that we've done our duty and reviewed what we're talking about, let's jump into some
practical use cases.

### Example 1: Combining spritesheets

When I first learned the news about `.flatMap()` coming to JS, I enthusiastically passed it
along to a coworker of mine with whom I usually talk about this sort of stuff. He's a more
recent developer, previously a tech writer, who entered into the web dev world by becoming a
fully self-taught PHP developer. (Major props to him and anyone else who takes this route, by
the way, especially later in life.) This coworker is not the sort of person who keeps up with
all the trends and patterns in the dev world (*engage snooty scoffing*), so he wasn't familiar
with `flatMap()` or `flatten()` as concepts, and was... underwhelmed. His response was sort of
a "huh, okay, and why do I care about that again? Seems interesting, but when would I want it?"

So I thought up an example that catered to his interests. This coworker is a
weekends-and-evenings indie game developer, so he's spent a lot of time working with sprites
and spritesheets. We've had lots of conversations about how best to serve spritesheets, when to
combine them into larger sheets, etc. So here's the example I presented to him:

Suppose you've created a spritesheet for monster in your game. Now you need to pack the frames
of your various separate spritesheets into a single larger `Image` file in an efficient way. Lots
of libraries exist to do this sort of work, so let's imagine one up and assume you're importing
two functions from it:

* `getFrames({img: Image, frameWidth: Number, frameHeight: Number}):Image[]` - accepts a
  spritesheet `Image` and the dimensions of each frame in that `Image` and splits the sheet into
  an array of individual frame `Image`s

* `makeSpritesheet(Image[]):Image` - accepts an array of `Image` objects containing frames
  and efficiently packs them into a single large spritesheet `Image` 

Now suppose you've got an array of spritesheet `Image`s loaded up in memory. You'll need to
call `getFrames()` for each spritesheet, which will return an array of `Image` objects
containing the frames for that sprite, then you'll need to combine all those `Image` arrays into
a single array to pass to `makeSpritesheet()`.

Without using `.flatMap()`, you'd need to do something like this:

```javascript
function combineSpritesheets(spritesheets) {
  let allFrames = []
  spritesheets.forEach(sheet =>{
    allFrames = allFrames.concat(getFrames(sheet))
  })
  return makeSpritesheet(allFrames)
}
```

That's *okay*, but it's a little less functional than it could be, and it feels a little bit
brittle. You could use `.reduce()` to clean things up a little:

```javascript
function combineSpritesheets(spritesheets) {
  return (
    makeSpritesheet(spritesheets)
      .map(getFrames)
      .reduce((all, frames) => all.concat(frames), [])
  )
}
```

That's definitely nicer, but it's still a bit verbose for my taste. Now, once you have
`.flatMap()`, it all becomes much cleaner:

```javascript
function combineSpritesheets(spritesheets) {
  return makeSpritesheet(spritesheets.flatMap(getFrames))
}
```
That's it! One line! How elegant is that? And in fact, we can do one better and make it an
arrow function to get a true one-liner:

```javascript
const combineSpritesheets = spritesheets => makeSpritesheet(spritesheets.flatMap(getFrames))
```

Simple, elegant, and readable. Each call to `getFrames()` returns an array of `Image` objects.
and `.flatMap()` concatenates these arrays together as it goes. Once all sheets have been
split, a single array containing all the frames from all the sheets is passed to
`makeSpritesheet()`, and we've got ourselves a combined sheet.







## Further reading

- [Jake Archibald's Tweet](https://twitter.com/jaffathecake/status/961585899275542528)
- [The WebKit changeset he linked that implements `Array.prototype.flatten()` and
  `Array.prototype.flatMap()`](https://trac.webkit.org/changeset/228266/webkit)
- ["Functional pattern: flatMap" - 2ality](http://2ality.com/2017/04/flatmap.html)



---
title: A surprising behavior of React's useEffect
layout: post
---

Since React 16.8 came out earlier this week, I've been playing around with hooks in personal projects so that I can better understand their ins and outs.

Many of the hooks that accept callbacks also take an optional array of values that are used for optimization purposes:
if none of the values in the array change between renders, the callback is skipped.

The [docs](https://reactjs.org/docs/hooks-reference.html#useeffect) contain a tantalizing hint about the future:

> The array of inputs is not passed as arguments to the effect function.
> Conceptually, though, that’s what they represent:
> every value referenced inside the effect function should also appear in the inputs array.
> In the future, a sufficiently advanced compiler could create this array automatically.

Until "the future" comes, determining which values should appear in the inputs array seems like a really tedious
(and error prone!) code review task,
so I tried writing a utility method to remove some of the "sharp edges":

<script src="https://gist.github.com/david-mitchell/6ee451a559c57246259a3c96ee549370.js"></script>

With this utility method, the values *are* passed to the callback.
If the callback is defined outside of the component, then it becomes *impossible* for it to use any unlisted values,
making code review much simpler.

Unfortunately, it has a critical flaw when used with refs.

Suppose I have a component that needs to attach an event handler directly to a DOM node:

<script src="https://gist.github.com/david-mitchell/dbbff27a8b7ac25bf6530a0052ed13db.js"></script>

*Aside: that this is more than just a hypothetical example: React attaches all event handlers at the document level.*
*One unfortunate consequence of this is that Chrome will force any touch event handlers to be passive.*
*This is undesirable for things like drawing controls which may need to call `preventDefault` to stop undesirable scrolling.*

As written, this effect will *never* be attached to the `canvas` element unless an ancestor of `CanvasElement` triggers a re-render.

This is the case because the value provided to `addTouchListener` gets bound at render time:
*before* `canvasRef.current` is set to an actual `canvas` element.
It's not until the second render--when `canvasRef.current` is set to the value from the first render--that `addTouchListener` is called with a DOM element.

This leads to another interesting discovery. Suppose we abandon `useBoundEffect` so that we can pass the correct value to `addTouchListener`:

<script src="https://gist.github.com/david-mitchell/f4abe8775fb496e9cd4b3bfe02ea7d7b.js"></script>

In this version, the effect attaches the event handler after the first render, as desired, but it *also* cleans up and re-attaches on the *second* render.
This happens for the same reason that the event handler was not attached the first time: the initial value of `canvasRef.current` is what is used to determine
whether the effect needs to be re-run.

React itself *could* provide a solution to this by allowing the second argument to be specified as a function that returns an array, instead of the array itself.
If React called this function immediately before applying the effect, it would enable a much more precise calculation of whether it needs to be re-applied.

On the other hand, I'm not keen to wait on such a feature, so I decided to see if I could emulate it.
This is what I came up with:

<script src="https://gist.github.com/david-mitchell/a9f9586aff7dd8b63b02ec2a1e868bf7.js"></script>

In this version, we have to use *two* effects. The first runs on every render to determine whether the inputs have changed.
The second is necessary to clean up when the component is unmounted: it runs only on the first render and sets up a cleanup method.

*Aside #2: if you try to use this in your code and supply the second argument in the form `() => [value1, value2]`, you may get an error*
*from the Typescript compiler complaining that the return type is not compatible with the expected type.*
*This happens because Typescript is really conservative about inferring arrays as tuples.*
*If you want to avoid adding a bunch of explicit types everywhere,*
*you can define a method like this to give the compiler the necessary hint:*

<script src="https://gist.github.com/david-mitchell/6b4917780cee8dc910848214bc6d2317.js"></script>

*With this in place, the second argument would change to the form `() => createTuple(value1, value2)`.*
*I'm hopeful that future versions of Typescript will make this simpler, as there are a number of GitHub Issues related to tuple inference;*
*on the other hand, there are no concrete proposals, so I'm not holding my breath.*

# React Fiber Architecture

## Introduction

**React Fiber is an ongoing reimplementation of React's core algorithm**. It is the culmination of over two years of research by the React team.
React uses a fiber algorithm to update **virtual DOM**

*The goal of React Fiber is to increase its suitability for areas like animation, layout, and gestures.* Its headline feature is **incremental rendering**: the ability to split rendering work into chunks and spread it out over multiple frames.

**Overall comparing the two trees ie, Actual DOM and Virtual DOM and accordingly, accommodate changes in actual DOM.**

**Other key features include the _ability to pause, abort, or reuse work as new updates come in; the ability to assign priority to different types of updates_;** and new concurrency primitives.

## HYDREATION

## What is reconciliation?

<dl>
  <dt>reconciliation</dt>
  <dd>The algorithm React uses to diff one tree with another to determine which parts need to be changed.</dd>

  <dt>update</dt>
  <dd>A change in the data used to render a React app. Usually the result of `setState`. Eventually results in a re-render.</dd>
</dl>

The central idea of React's API is to think of updates as if they cause the entire app to re-render. This allows the developer to reason declaratively, rather than worry about how to efficiently transition the app from any particular state to another (A to B, B to C, C to A, and so on).

**Actually re-rendering the entire app on each change only works for the most trivial (minor) apps; in a real-world app, it's prohibitively costly in terms of performance. React has optimizations which create the appearance of whole app re-rendering while maintaining great performance.** The bulk of these optimizations are part of a process called **reconciliation**.

*Reconciliation is the algorithm behind what is popularly understood as the "virtual DOM."* A high-level description goes something like this: when you render a React application, a tree of nodes that describes the app is generated and saved in memory. This tree is then flushed to the rendering environment — for example, in the case of a browser application, it's translated to a set of DOM operations. When the app is updated (usually via `setState`), a new tree is generated. **The new tree is diffed with the previous tree to compute which operations are needed to update the rendered app.**

The key points are:

- Different component types are assumed to generate substantially different trees. React will not attempt to diff them, but rather replace the old tree completely.
- Diffing of lists is performed using keys. Keys should be "stable, predictable, and unique."

### Reconciliation versus rendering

The DOM is just one of the rendering environments React can render to, the other major targets being native iOS and Android views via React Native. (This is why "virtual DOM" is a bit of a misnomer.)

The reason it can support so many targets is because React is designed so that reconciliation and rendering are separate phases. The reconciler does the work of computing which parts of a tree have changed; the renderer then uses that information to actually update the rendered app.

This separation means that React DOM and React Native can use their own renderers while sharing the same reconciler, provided by React core.

Fiber reimplements the reconciler. It is not principally concerned with rendering, though renderers will need to change to support (and take advantage of) the new architecture.

### Scheduling

<dl>
  <dt>scheduling</dt>
  <dd>the process of determining when work should be performed.</dd>

  <dt>work</dt>
  <dd>any computations that must be performed. Work is usually the result of an update (e.g. <code>setState</code>).
</dl>

> In its current implementation React walks the tree recursively and calls render functions of the whole updated tree during a single tick. However in the future it might start delaying some updates to avoid dropping frames.
>
> This is a common theme in React design. Some popular libraries implement the "push" approach where computations are performed when the new data is available. React, however, sticks to the "pull" approach where computations can be delayed until necessary.
>
> React is not a generic data processing library. It is a library for building user interfaces. We think that it is uniquely positioned in an app to know which computations are relevant right now and which are not.
>
> If something is offscreen, we can delay any logic related to it. If data is arriving faster than the frame rate, we can coalesce and batch updates. We can prioritize work coming from user interactions (such as an animation caused by a button click) over less important background work (such as rendering new content just loaded from the network) to avoid dropping frames.

### The key points are:

*- In a UI, it's not necessary for every update to be applied immediately; in fact, doing so can be wasteful, causing frames to drop and degrading the user experience.*
*- Different types of updates have different **priorities** — **an animation update needs to be completed more quickly than, say, an update from a data store.***
*- A push-based approach requires the app (you, the programmer) to decide how to schedule work. A pull-based approach allows the framework (React) to be smart and make those decisions for you.*

React doesn't currently take advantage of scheduling in a significant way; an update results in the entire subtree being re-rendered immediately. Overhauling React's core algorithm to take advantage of scheduling is the driving idea behind Fiber.


## Future sections

That's all there is for now, but this document is nowhere near complete. Future sections will describe the algorithms used throughout the lifecycle of an update. Topics to cover include:

- how the scheduler finds the next unit of work to perform.
- how priority is tracked and propagated through the fiber tree.
- how the scheduler knows when to pause and resume work.
- how work is flushed and marked as complete.
- how side-effects (such as lifecycle methods) work.
- what a coroutine is and how it can be used to implement features like context and layout.

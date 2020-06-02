# Scheduling Tasks in JavaScript

While investigating some performance issues with intense data loading at Robinhood, we noticed some interesting performance implications with various ways to schedule tasks in JavaScript. This document compiles the list of task scheduling functions and their behavioral differences.

## Tasks vs microtasks

Understanding the difference between tasks and microtasks would be really helpful to follow the rest of the article. A good read can be found at [here](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)

## setTimeout / setInterval

These two functions offer the typical method to append a task to the task queue.

> An interesting note is setTimeout(callback, 0) doesn't execute the callback immediately as it is [clamped](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout#Reasons_for_delays_longer_than_specified) to a minimum wait time of 4ms.

## queueMicrotask or promises

Append to microtask queue. The microtasks are usually executed immediately following the current task.

> The microtask has one special feature that would potentially cause performance issue. If a mirotask adds more microtasks to the queue, the newly added microtasks execute BEFORE the task next task is run. In practice, if a microtasks (promise) keep queuing more microtasks, the main thread would be blocked.

## requestIdleCallback

A ["silver bullet"](https://developers.google.com/web/updates/2015/08/using-requestidlecallback) to schedule background tasks of various priorities. RequestIdleCallback will schedule work when there is free time at the end of a frame, or when the user is inactive.

By specifying a [timeout](https://developers.google.com/web/updates/2015/08/using-requestidlecallback#guaranteeing_your_function_is_called), it gives the flexibity to make sure the callback is executed after the specified time.

> In practice, requestIdleCallback is a better fit to schedule background tasks than microtasks because queuing more idle callbacks won't overwhelm the main thread like microtasks.

## requestAnimationFrame

RequestAnimationFrame callback is called right before the browser paints a new frame. It is perfect for animation but also useful for [throttling](https://developer.mozilla.org/en-US/docs/Web/API/Document/scroll_event#Scroll_optimization_with_window.requestAnimationFrame) some high rate events like scroll.

One caveat with requestAnimationFrame is its callbacks are paused in most browsers when running in background tabs or hidden.

> Also in Firefox's [performance tips](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Performance_best_practices_for_Firefox_fe_engineers), it's fine perform DOM writes inside rAF, but not recommended to put queries for layout or style information.

## setImmediate

While this function does what it name says, it's considered a non-standard feature among browsers. It is very useful to breakup long running tasks in node.

Here's an good [answer](https://stackoverflow.com/questions/15349733/setimmediate-vs-nexttick#:~:text=Use%20setImmediate%20if%20you%20want,already%20in%20the%20event%20queue.&text=nextTick%20to%20effectively%20queue%20the,after%20the%20current%20function%20completes.) for setImmedaite vs nextTick in node.

## React Scheduler

React actually has a more sophisticated scheduler with more controls over requestAnimationFrame and requestIdleCallback [here](https://github.com/facebook/react/blob/43a137d9c13064b530d95ba51138ec1607de2c99/packages/react-scheduler/src/ReactScheduler.js).

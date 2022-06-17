---
title: Comparing Svelte Reactivity Options
date: '2022-01-06'
tags:
---

<div class="message"> The Svelte framework has several reactivity features: Assignments, Statements, Stores and Component Events. This post summarises the options and explores when you might use each one.
</div>

## Reactivity

Reactivity is a coding "style" where when some data changes, other dependent data also gets updated, automatically. Perhaps the most common occurrence of this is in spreadsheets where changing a cell value causes all referencing cells to also update. This is often seen as a "push" semantic, compared to a "pull" sematic of calling a function.

Reactivity is often implemented using a version of the "publish / subscribe" (pubsub) pattern, where any number of subscribers can be updated with changes broadcast from a publisher.

Event systems such as DOM events provide one form of highly decoupled reactivity where an event (or message) is dispatched from one element and other elements may handle it. In the DOM the handler is a callback function.

Reactivity is not usually built into programming languages. JavaScript requires a library or framework feature. However, there is a Stage One TC39 Proposal to add an [Observable](https://github.com/tc39/proposal-observable) type to JavaScript. This is based on the [RxJS](https://rxjs.dev/) Reactive Extensions Library for JavaScript, which enables a style of programming called Functional Reactive Programming (FRP) or "streams". Another popular reactive library for JavaScript is [MobX](https://mobx.js.org/getting-started.html) state manager.

FRP is often referred to by the shorter Reactive Programming (RP) which is actually technically a bit different. FRP, like other functional programming styles, involves data flowing through small functions such as `map` and `reduce`. Code thus consists of declarative chains of expressions, compared to imperative lists of statements or object oriented methods operating on private data.

## Svelte Reactivity

One of the highly satisfying features of the [Svelte](https://svelte.dev/) web app development framework is that reactivity is baked in (unlike React). Any **assignment** to a local variables is reactive and will cause the component to be re-rendered when the value changes.

```javascript
<script>
	let count = 0;

	function handleClick () {
		count = count + 1;
	}
</script>

<button>
	Clicked {count} {count === 1 ? 'time' : 'times'}
</button>

```

There are also three more explicit reactive features and while the excellent Svelte documentation describes each, it may not be obvious which one to use. None of these features go quite as far as FRP. But, they are nevertheless extremely powerful and easy to use when creating interactive web apps with Svelte.

Note that as with other JavaScript reactive libraries the `$` symbol is conventionally used to identify reactive elements.

### Reactive Statements

Statements can be marked as being reactive. These can be single line or block statements. The statement will be revaluated whenever any variable or properties directly referenced in it are changed. As with simple assignments, changes to will cause the enclosing component to be re-rendered.

```javascript
<script>
	let count = 0;
	$: doubled = count * 2;

	function handleClick() {
		count += 1;
	}
</script>

<button on:click={handleClick}>
	Clicked {count} {count === 1 ? 'time' : 'times'}
</button>

<p>{count} doubled is {doubled}</p>
```

This form of reactivity is marked with the rarely used JavaScript [label](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label) syntax using a variable name of `$`, ie lines of code that start with `$:`. To reference the value in a component the variable name is used directly, without any `$`.

Updates are synchronous such that code changing the source variable will return after all the referencing reactive statements are also updated.

Reactive assignments and their references are useful within code in a single file such as a Svelte component. For example, a event handler declared in the HTML section may update a variable.  That variable can be referenced in a reactive statement in the script section to derive a new value. Then, the new value can  referenced in the HTML as with a simple variable assignment. Then, whenever the event occurs the HTML content will be updated to match the new computed value.

See the [tutorial](https://svelte.dev/tutorial/reactive-assignments) and [docs](https://svelte.dev/docs#component-format-script-2-assignments-are-reactive) for details.

### Reactive Stores

Svelte Stores are reactive and use the Observable pattern. They are similar to Observable types (see above). In Svelte the store is an observable which broadcasts changes to any code that is subscribed to it.

Stores are created using library functions and there are three types: `readable`, `writable` and `derived`. Readables are sources of data streams with sequential values being created in a callback. Writables may also be updated by external code using the `set` and `update` methods. Derived stores provide composition by computing and emitting values based on those from other stores when those are updated. Stores may be subscribed to by calling the `subscribe` method or the Svelte compiler provides syntactic sugar in the form of prefixing the store name with a `$`.

```javascript
<script>
import { readable } from 'svelte/store';

export const time = readable(new Date(), function start(set) {
	const interval = setInterval(() => {
		set(new Date());
	}, 1000);

	return function stop() {
		clearInterval(interval

const formatter = new Intl.DateTimeFormat('en', {
		hour12: true,
		hour: 'numeric',
		minute: '2-digit',
		second: '2-digit'
	});
</script>

<h1>The time is {formatter.format($time)}</h1>
```

Updates can be synchronous or asynchronous depending on how the updating function is coded.

Stores are useful if you need asynchronous reactivity or want to share updates between code in many places, including components not in a parent-child relationship. For example, a component event handler can update a writable store and all subscribed components will be updated. They can also be used outside components. While Svelte doesn't conceptually deal with FRP or streams it is easy to to integrate other observable libraries like RxJS as the APIs are standardised.

See the [tutorial](https://svelte.dev/tutorial/writable-stores) and [docs](https://svelte.dev/docs#run-time-svelte-store) for details.

### Component Events

Svelte Component events provide a form of pubsub reactivity between child components and their parents. Like the [DOM custom event](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent) object, which they use, they can be simple triggers or also pass data values (aka details). Unlike DOM events they do not use PostMessage to propagate around the DOM hierarchy, Rather, they directly call the component's event handler for the event.

Events are created using the `createEventDispatcher` function. They are handled using the Svelte DOM event handler syntax `on:event-name={handler}`. A custom event name is supplied rather than a DOM event name. The handler will receive the event including any extended details provided by the dispatcher. A form of the event handler syntax is used to pass the event on to the parent without any processing.

```javascript
# File Inner.svelte
<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function sayHello() {
		dispatch('message', {
			text: 'Hello!'
		});
	}
</script>

<button on:click={sayHello}>
	Click to say hello
</button>

# File App.svelte
<script>
	function handleMessage(event) {
		alert(event.detail.text);
	}
</script>

<Inner on:message={handleMessage}/>
```

Events are synchronous. The component event handlers are called directly when the message is dispatched.

Component Events are useful when you want to use a decoupled event or message pattern, including to pass up the component tree.

See the [tutorial](https://svelte.dev/tutorial/component-events) and [docs](https://svelte.dev/docs#run-time-svelte-createeventdispatcher) for details.

## Further Info

- More details and examples of using `$` in [The many meanings of $ in Svelte](https://geoffrich.net/posts/svelte-$-meanings/) by Geof Rich

{:title "Re-frame: side effects as data"
 :layout :post
 :klipse true
 :tags  ["re-frame" "reagent" "clojurescript" "klipse"]}

Re-frame has a simple but powerful architecture that enables you to express yourself
in pure data structures even with side-effect heavy code.
In this post I will show a simple example of the patterns used.

### Re-frame

I'm going to assume that you have some basic knowledge about [Re-frame],
the very popular lightweight application framework for [Reagent] applications. The architecture is very
similar to React Redux, in that you have *event handlers / reducers* that take the current version of your
state as an argument and returns a transformed version of the state.

### Data first
Clojure is all about data. It has a huge standard library that allows you to manipulate your immutable data structures
in every possible way.
This is a great tool for reducers when doing their data crunching.

### How about those nasty side effects?
Any serious app needs to do HTTP, navigation, cookies, local storage, and other kinds of effects that are not about manipulating
some data structure. It turns out that the re-frame developers did some very serious thinking about this. They came up with
the concept of *pluggable effects*. Instead of executing functions you will return data that tells some other party how to
get the job done.

Rather than trying to explain this in words, I will use the amazing KLIPSE
plugin to let you play around with these concepts yourself! Feel free to edit the code below to see what happens.
### Boot it up!
First, we need to require re-frame, and transitively reagent, into this page

```klipse-cljs
(require '[re-frame.core :as re-frame])
```

#### Introducing events

This is a typical re-frame event handler. We use `reg-event-fx` to get access to both incoming and outgoing side effects.


```klipse-cljs

(re-frame/reg-event-fx
:show-message
    (fn [incoming-effects [event-key message]]
        (js/alert (str "I was DIRECTLY asked to print this: " message))))
nil
```

As you can see, the handler shows the provided message in a native javascript `alert` and returns nil (meaning nothing changed and there is nothing more that the framework needs to take care of).
Click the button below to trigger the event and see the effect.

```klipse-reagent
[:button
    {:on-click (fn [e] (re-frame/dispatch [:show-message "42"]))}
    "Click me to alert something!"]
```

For many cases, this will be just fine. But in a more complex system with less visible side effects, you quickly lose track
of where you are when debugging. One thing that really helps in these cases is a full and complete overview of the code paths, after the fact.
If we execute our side effects inside the black boxes that functions are, we have no such overview.

### Effects as data

If we manage to express our side effects as data, this overview becomes trivial to make. To do this we need a couple of building
blocks. First out are pluggable effects. You are free to create your own types of side effects, using re-frame's `reg-fx` function.

```klipse-cljs
(re-frame/reg-fx
    :alert
    (fn [message]
        (js/alert (str "I was INDIRECTLY asked to print this: " message))))
nil
```

Nothing too fancy there. We are just introducing a level of indirection, letting the framework do the work of connecting
our data to the actual effect.

### Logging

To get the full trace that we want, we need a way to inspect the return value of this handler after it executed.

Re-frame uses [interceptors] for cross-cutting concerns like this. The one below runs after the event handler.
Through its `context` parameter it has access to all metadata, including the return value of the handler. The `effects` function
gives us the handler return value.

```klipse-cljs

(require '[re-frame.interceptor :refer [->interceptor get-effect]])

(def debug-interceptor
  (->interceptor
    :id     :log-effects
    :after  (fn [context]
             (let [effects (get-effect context)
                   event (re-frame/get-coeffect context :event)]
               (if (seq effects)
                 (js/alert (str "Event " event " caused side effect " effects)))
               context))))
```

Printing the event effect data to the console would probably be a much better idea, but since this
is a demo inlined in a blog, we'll use an alert box.

### A "pure" side effecting handler

Now we have what we need to be able to inspect or test our side effect. We inject our interceptor into the event handler when
registering it.

```klipse-cljs
(re-frame/reg-event-fx
    :show-message-with-indirection
    [debug-interceptor]
    (fn [_ [_ message]] {:alert message}))
nil
```

And finally, here's a snippet that uses our new handler. Click the button to observe
the alert box with the event data, prior to the actual effect.

```klipse-reagent
[:button
    {:on-click (fn [e] (re-frame/dispatch [:show-message-with-indirection "42"]))}
    "Click me to alert something!"]
```

The difference between `(js/alert message)` and `{:alert message}` might seem insignificant. But having the trace of data
at your fingertips makes a world of a difference when you're stuck trying to figure out why your chain of HTTP requests
 has stalled. Unit testing also becomes trivial, as opposed to sniffing the presence of an alert box.

### Event ping pong

I'm not super happy about the callback style of re-frame. Every time you need to do something asynchronous like HTTP, you
need to name the event handler that should receive the callback. It very quickly turns into something that could be called
 "event ping pong" or even "callback hell". [keechma] is a very interesting alternative framework, the [pipelines] are particluarly interesting.
 But keechma does not have the same focus on effects as data.

I would love to see something like the keechma pipelines for re-frame, and I'm experimenting to see if I can find a nice
and practical solution.

If you find this subject interesting hit me up on [reddit] or [twitter]!

[interceptors]: https://github.com/Day8/re-frame/blob/master/docs/Interceptors.md
[re-frame]: https://github.com/Day8/re-frame
[reagent]: https://reagent-project.github.io/
[keechma]: https://keechma.com/
[pipelines]: https://keechma.com/news/introducing-keechma-toolbox-part-1-pipelines/
[reddit]: https://www.reddit.com/r/Clojure/comments/6k8ylv/reframe_side_effects_as_data/
[twitter]: https://twitter.com/ingesol/status/880443720977588224
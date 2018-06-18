{:title "Kee-frame controller tricks"
 :layout :post
 :tags  ["clojurescript" "re-frame" "kee-frame"]}

##  Controllers
The [controller rules](https://github.com/ingesolvoll/kee-frame#controller-state-transitions), taken from [keechma](https://keechma.com/),
are centered around pure route data. That's really powerful, as you can use the full
power of Clojure to get what you want from your controllers. Often the solution is dead simple, but it's not always easy to spot.
This is a guide to the most useful tricks, it will be expanded as new ones appear!

## How to trigger an event once, at startup

So you want something to happen only once, but immediately. Things like:
* Fetching initial data from the server
* Start a polling loop
* Letting the user know we are loaded and ready to go 

What you need is a `:params` function that triggers start when invoked for the first time, and then always returns the 
same result as the first invocation. That sounds like a plain Clojure function that we all know:

```clojure
{:params (constantly true) ;; true, or whatever non-nil value you prefer
 :start  [:call-me-once-then-never-again]}
```

## How to restart a controller on every route change

Maybe you want to store a trail of breadcrumbs, maybe you want some logging done. Either way, this one is also quite simple.

For this case you need a `:params` function that returns a new unique result for every new unique route data. What tiny but
familiar Clojure function could we use to achieve that?

```clojure
{:params identity
 :start  [:log-user-activity]}
```

## Getting weird

As you can see, most things with controllers are quite simple and use plain Clojure. Just for fun, let's have a look
at a controller that triggers randomly:

```clojure
{:params #(rand-nth [nil % % %])
 :start  [:will-receive-the-route-data-quite-often-but-not-always]}
```

That's it for now. Please report your useful tricks at [kee-frame issues](https://github.com/ingesolvoll/kee-frame/issues) and
I'll add them here!
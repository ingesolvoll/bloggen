{:title "Kee-frame: getting re-frame right, faster"
 :layout :post
 :tags  ["clojurescript re-frame kee-frame"]}

Most developers will claim that TDD is part of their daily routine. But most of us fail to mention
our actual favorite methodology, SDD.

Stackoverflow Driven Development is a very good thing. Sharing knowledge and experience moves everything forward, and we
should keep going. But some issues are too large or too important to quick-fix through an upvoted stackoverflow answer.
The "best practice" might not be the best for you.

## How to load data from the server when navigating to a view?
One very common Stackoverflow question for React developers is how to get React/Redux/Re-frame/etc to load data from the
server at the correct time (when your view component is about to be rendered). You will usually be advised to use the
React lifecycle methods. In re-frame it looks like this:

```clojure
(defn customer-list [_]
  (r/create-class
    {:component-did-mount #(dispatch [:fetch-customers])
     :reagent-render      (fn [customers]
                            (if (seq customers)
                                [:ul (map (fn [customer]
                                    [:li (:name customer)]))]
                                [:div "Loading customers"]))}))
```

Is this a good pattern for your app? It depends. If the only tool you have at hand is React itself, this "best practice"
is in fact a quite decent practice, and it works. But it does come with a few disadvantages:

* The React class verbosity hides the intent of your code
* Your view code is no longer a pure function of your data, it has side effects.
* Having your data loading events littered around in your UI code makes them very hard to find and follow. Especially as your app grows larger.
* You might not want to contact the server on every single UI tweak you make, the data is already there, you don't need another copy.

## Optimizing for development

Clojure developers know what a smooth development environment means for productivity and quality. It is not optional or nice to have, it is
something you gladly pay for up front. But the way you organize your code also affects your workflow in a big way. Figwheel is possible
because Clojure encourages you to push the mutable state towards the edge of your system, allowing most of your functions
to be pure and thereby easily reloadable.

Most people get that part right. But it gets trickier from there. On the client you might have things like websockets,
polling loops or web workers. When figwheel reloads, you need the lifecycle of those to be handled gracefully. It's not very productive to
end up with either one dead socket or 20 duplicated ones, forcing you to refresh your browser. You can get a long way using
[mount](https://github.com/tolitius/mount), but I'm not sure it was built for fine-grained control of timing.

Figwheel shines when it renders the exact same screen as before, only with your code changes updated. No extra duplicated go-loops,
no jumping to another tab in your app, no missing values in state. But all of this requires effort and design on your end,
figwheel only offers you a helping hand.

In general, setting up your client side routing in a way that gives you both good application structure and development
flow requires a lot of thinking and effort. It makes it unnecessarily hard to get started making SPAs in Clojurescript. And
if you skip this work in the beginning, you build a lot of complexity and pain for the longer run.

##
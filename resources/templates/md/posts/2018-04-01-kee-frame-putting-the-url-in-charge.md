{:title "Kee-frame: Putting the URL in charge"
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

This "best practice" works fairly well, but it has a couple of disadvantages:

* The React class verbosity hides the intent of your code
* Your view code is no longer a pure function of your data, it has side effects.
* Having your data loading events littered around in your UI code makes them very hard to find and follow.
* It's unpredictable and tricky during development if you're not a React lifecycle expert.

## Optimizing for development

Clojure developers know what a smooth development environment means for productivity and quality. It is not optional or nice to have, it is
something you gladly pay for up front. But the way you organize your code also affects your workflow in a big way. Figwheel is possible
because Clojure encourages you to push the mutable state towards the edge of your system, allowing most of your functions
to be pure and thereby easily reloadable.

Most people get that part right. But it gets trickier from there. On the client you might have things like
***websockets,polling loops or web workers***. When figwheel reloads, you need the lifecycle of those to be handled gracefully. It's not very productive to
end up with either one dead socket or 20 duplicated ones, forcing you to refresh your browser. You can get a long way using
[mount](https://github.com/tolitius/mount), but I don't think it's the right tool for this.

Figwheel shines when it renders the exact same screen as before, only with your code changes updated. No extra duplicated go-loops,
no jumping to another tab in your app, no missing values in state. But for this to work you need to set up a lot of non-trivial stuff.
It makes it unnecessarily hard to get started making SPAs in Clojurescript. And if you skip this work in the beginning, you build a
lot of complexity and pain for the longer run.

## Pushing you in the right direction
The goal of [kee-frame](https://github.com/ingesolvoll/kee-frame) is to make it easy to get started quickly with
a nice re-frame setup, while providing tools that make it easier to do the right thing in common scenarios. It does so by
***putting the URL in charge***.

What does that mean, putting the URL in charge? It means putting the ***web*** back in ***web app*** . A friendly web app should:
* Go to the most recently viewed page when browser's back button is clicked
* View the exact same content when the browser's refresh button is clicked
* Make all app URLs bookmarkable (a derived property from the previous point)

These are usability features, but they also help you towards a cleaner design for your SPA. If the view
decides when to load some data, your view and your model are tightly connected. But if you put the URL in charge, the view
and the model are disconnected and the URL controls them both.

***Kee-frame doesn't force you to do this, it just provides you with the tools that make the best solutions the easiest ones.***

## A kee-frame app is a re-frame app

Most of your re-frame code can stay just the way it is within kee-frame. Kee-frame just provides a smoother startup
experience. With the code below, you skip the ceremony of setting up routing and rendering:

```clojure
(kee-frame/start! {:routes         ["/"   {""           :index
                                           "customers"  :customers}]
                   :initial-db     {:loading?           false
                                    :logged-in-user     nil
                                    :customers          nil}
                   :root-component [main-view]})
```

What we have now is a routing system that shields you from the dirty details
of URLs and navigation. Your app ***only operates on route data*** . The rest of kee-frame builds on this core feature,
providing the following:

* Controllers use route data to trigger re-frame events at the right time
* When you need a `<a href="something">`, kee-frame generates that link for you.
* If you want to navigate the browser to a different URL in a re-frame event, kee-frame has a side-effect handler for that. Based on pure data of course.
* A reagent component to choose the right view for your route data.

## URL-driven data loading (UDDL)

Ok, that's not a real acronym. But let's revisit the data loading example. With kee-frame, this is how you do it:

```clojure
(kee-frame/reg-controller   :customers
                            {:params (fn [{:keys [handler]}]
                                       (when (= handler :customers) true))
                             :start  (fn [ctx _]
                                       [:fetch-customers])})

(defn customer-list [customers]
    (if (seq customers)
        [:ul (map (fn [customer]
            [:li (:name customer)]))]
        [:div "Loading customers"]))

```

The reagent component is very simple now, just a pure function of its parameter. You could easily sprinkle some unit tests
on that.

The `params` function is invoked every time the route changes. When it returns a non-nil value different from
the previous value, the `start` function is invoked. If the `start` function returns a re-frame event vector, it does a `dispatch`.

## Profit!

So, what did we achieve? A few things, but they're significant:
* Pure UI components
* High cohesion as you can put your controller next to your data loading events and get a clear overview of your data flow.
* Timing of events is simple and intuitive: everything happens on route change
* System performs predictably during development. Re-render is just re-render. Side effects happen on route change, not on code reload.

## More info

This blog post is a minimal and simplified description of kee-frame. Go to the [kee-frame github page](https://github.com/ingesolvoll/kee-frame) for a more in-depth introduction.
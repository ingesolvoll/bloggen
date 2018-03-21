{:title "Clean up your data loading with kee-frame"
 :layout :post
 :tags  ["clojurescript re-frame kee-frame"]}

If you're like me, TDD is part of your daily routine. But you are probably also doing a lot of SDD
(Stackoverflow Driven Development).

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
                                [:div "Loading customers"])}))
```

I've used this pattern a lot, and it works. `[:fetch-customers]` calls the server and puts the returned data into the app db
where the view can find it.

But it has some important disadvantages.

* The React class verbosity hides the intent of your code
* Your view code is no longer purely declarative
* Your data graph might not match very well with your UI graph.
* Having your data loading events littered around in your UI code makes them very hard to find and follow.

## Code reloading

Another issue to consider is your figwheel workflow. Let's pretend you are currently working on this customer list component, saving your
file every 10 seconds to see how it's shaping up. Every time you trigger another compile and render, an AJAX request is triggered by
the `:fetch-customers` event. To me this pattern is purely accidental and it does not make sense. It could also slow down your
feedback loop.

## URL is boss


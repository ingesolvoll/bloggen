{:title "Client side routing done right"
 :layout :post
 :tags  ["clojurescript re-frame kee-frame"]}

Most developers will claim that TDD is part of their daily routine. But most of us fail to mention
our actual favorite methodology, SDD. Stackoverflow Driven Development has become the number 1 thing to learn if you
want to be successful making web applications.

This is a very good thing. Sharing knowledge and experience moves everything forward, and we should keep going. But some
issues are too large or too important to quick-fix through an upvoted stackoverflow answer. The "best practice" might
not be the best for you.

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

## Optimizing for development




## URL is boss

To me,
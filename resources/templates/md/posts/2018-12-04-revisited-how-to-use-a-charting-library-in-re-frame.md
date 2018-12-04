{:title "Revisited: How to use a charting library in reagent"
 :layout :post
 :klipse true
 :tags  ["reagent"  "klipse" "clojurescript"]}

## Improved advice
I wrote a [blog post](http://ingesolvoll.github.io/posts/2017-01-01-how-to-use-a-charting-library-in-reagent/) about this 
subject a couple of years ago and a lot of people read it. Unfortunately it contains
some advice that has been [proven wrong/not optimal](https://github.com/Day8/re-frame/blob/master/docs/Using-Stateful-JS-Components.md). 
The re-frame explanation of this concept is nice and understandable, but I wanted to make an interactive example so it's even clearer.

## Requirements
We'll be purely practical in this walkthrough, so let's get right to it.
```klipse-cljs
(require '[reagent.core :as r])
(require '[cljsjs.highcharts])
```

We'll use a regular reagent atom as our mutable data container. It could be anything with a "ratom" behaviour, like a 
re-frame subscription. This reference will hold the configuration and data for the chart.

```klipse-cljs
(def config-atom (r/atom nil))
```

## Inner/outer

Now for the interesting bit. The trick here, as described in the re-frame docs, is to split the integration in two.
- An outer component that closes over the Clojure reactive reference
- An inner component that accepts a React props map derived from the reactive reference


##The inner component

For this example the `mount-chart` and `update-chart` functions are identical. In less trivial cases they are likely different, and you
probably want to call the `update-chart` function at the end of your `mount-chart` function.

```klipse-cljs

(defn mount-chart [comp]
    (.chart js/Highcharts (r/dom-node comp) (clj->js (r/props comp))))

(defn update-chart [comp]
    (mount-chart comp))

(defn chart-inner []
  (r/create-class
    {:component-did-mount   mount-chart
     :component-did-update  update-chart
     :reagent-render        (fn [comp] [:div])}))
```

##The outer component

This one handles the mutable state from the clojurescript side. Dereference the atom/subscription/reaction and pass the
necessary information on to the inner component. The inner component will re-render on changes to these props, giving 
us the behaviour we want.

```klipse-cljs
(defn chart-outer [config]
    [chart-inner @config])
```

##The configuration

The following renders a simple chart below. Feel free to play around with the config to see what happens!

```klipse-cljs
(reset! config-atom {:chart {:type   :bar}
                             :title  {:text "Chart title here"}
                             :xAxis  {:categories ["Apples", "Bananas", "Oranges"]}
                             :yAxis  {:title {:text "Fruit eaten"}}
                             :series [{:name "Jane" :data [1, 0, 4]}
                                      {:name "John" :data [5, 7, 3]}]})
nil
```

```klipse-reagent
[chart-outer config-atom]
```
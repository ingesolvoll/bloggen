{:title "How to use a charting library in Reagent"
 :layout :post
 :klipse true
 :tags  ["reagent"  "klipse" "clojurescript"]}

# This post is outdated
The technique detailed in this blog post works. But there is a cleaner way, recommended by the re-frame team.

[Read the updated guide here](/posts/2018-12-04-revisited-how-to-use-a-charting-library-in-re-frame/)

### Charting in javascript
There are several high quality charting libraries made for the browser. I chose Highcharts for this demo. Highcharts is an excellent commercial charting library with both great looking charts and simple configuration. I'm going to use this [example] as a basis for this demonstration.

### This web page is interactive
I will be using the amazing [KLIPSE] plugin in this blog post. It turns all the code samples into live code that you can edit and experiment with.

First, we need to load Reagent, React and highcharts into the page. This should take about 5-10 seconds. After that, it will show the result `nil`, as expected for the `require` statement
```klipse-cljs
(require '[reagent.core :as r])
(require '[cljsjs.highcharts])
```

The config for our chart will reside in a reagent `atom`, allowing us to live-configure the chart later on.
```klipse-cljs
(def config-atom (r/atom nil))
```

Now we make the reagent component that will render our chart. We hook on to the React lifecycle event `component-did-mount` to render the chart when our component has been added to the DOM. We also include `component-did-update` to re-render the chart on your config changes.

```klipse-cljs

(defn render-chart-fn [config-atom]
  (fn [component]
    (.chart js/Highcharts (r/dom-node component) (clj->js @config-atom))))

(defn chart-ui [config-atom]
  (r/create-class
    {:component-did-mount (render-chart-fn config-atom)
      :component-did-update (render-chart-fn config-atom)
      :reagent-render (fn [config-atom]
        @config-atom ;; Dirty hack, so reagent will re-render this component when config changes
        [:div])}))
```

One major gotcha here is the `clj->js` part. Without it, you will be sending Clojurescript data structures to Highcharts, with weird error messages and painful meaningless debugging as a result. Observe the difference below

```klipse-cljs
(type {:this :is
  :not :javascript})
```

```klipse-cljs
(type (clj->js {:this :however
  :is :javascript}))
```

Let's make a basic config for our chart and put it in the atom

```klipse-cljs
(def default-config
  {:chart {:type :bar}
  :title {:text "Chart title here"}
  :xAxis {:categories ["Apples", "Bananas", "Oranges"]}
  :yAxis {:title {:text "Fruit eaten"}}
  :series [{:name "Jane" :data [1, 0, 4]}
          {:name "John" :data [5, 7, 3]}]})

(reset! config-atom default-config)
```

The end result is rendered below.

```klipse-reagent
[chart-ui config-atom]
```

You should try to edit the configuration. The chart will re-render immediately when you make changes. Try things like:

* Change the title
* Change the category names
* Change chart type from `:bar` to `:line`

As you can see, integrating any library in Reagent is fairly simple and concise. There are some pitfalls, some important ones have been covered here. The most significant one is the use of [externs] when you build your app with advanced optimizations. If you use [cljsjs], that won't be an issue. Chart libraries like highcharts, C3, D3 are on cljsjs.

[example]: http://www.highcharts.com/docs/getting-started/your-first-chart
[KLIPSE]: http://blog.klipse.tech/reagent/2016/12/31/reagent-in-klipse.html
[externs]: http://www.lispcast.com/clojurescript-externs
[cljsjs]: https://cljsjs.github.io/

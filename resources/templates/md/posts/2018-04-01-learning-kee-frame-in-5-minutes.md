{:title "Learning kee-frame in 5 minutes"
 :layout :post
 :tags  ["clojurescript" "re-frame" "kee-frame"]}

You are here, so you probably heard about the [kee-frame framework](https://github.com/ingesolvoll/kee-frame).
It's  built on re-frame, and should be easy to learn. Let's see if that's true! If you want to see the full context
for the code samples in this tutorial, they're all part of the [kee-frame demo app](https://github.com/ingesolvoll/kee-frame-sample)

## Installation

If you're starting a brand new project, I would recommend using a lein template, like this:

```clojure
lein new re-frame <your-project-name>
```

Once you have a working re-frame project, just add kee-frame as a dependency to your project:

```clojure
[kee-frame "0.2.1"]
```

Now run `lein figwheel` and we're good to go.

## Startup

The `start!` function hooks everything up, including **routing, spec-validation, debug logging, initial db and rendering** .

```clojure
(require '[kee-frame.core :as k])

(def routes ["" {"/"                       :live
                 ["/league/" :id "/" :tab] :league}])

(def initial-db {:leagues       nil})

(s/def ::league (s/keys :req-un [::caption ::id]))
(s/def ::leagues (s/nilable (s/coll-of ::league)))
(s/def ::db-spec (s/keys :req-un [::leagues]))

(k/start! {:debug?         true
           :routes         routes
           :initial-db     initial-db
           :root-component [main-panel]
           :app-db-spec    ::db-spec})
```

## Controllers

After running `start!`, we're ready to play with controllers. This one will dispatch the event `[:league/load 3]` when we're
at the url `/league/3`.

```clojure
(reg-controller :league
                {:params (fn [{:keys [handler route-params]}]
                           (when (= handler :league)
                             (:id route-params)))
                 :start  [:league/load]}) ;; Start can be either a function returning an event vector or just an event vector directly
```

The next controller starts and stops a polling loop when we enter and leave the live page.

```clojure
(reg-controller :live-polling
                {:params (fn [{:keys [handler]}]
                           (when (= handler :live) true))
                 :start  [:live/start]
                 :stop   [:live/stop]})
```

## Event chains

Chains are just regular re-frame FX events that are chained together by the framework. Let's first have a look at a very typical pair of events
loading some data over HTTP:

```clojure
(reg-event-fx :leagues/load
              (fn [_ _]
                {:http-xhrio   {:uri        "http://api.football-data.org/v1/competitions/?season=2017"
                                :on-success [:leagues/load-1]}}))

(reg-event-fx :leagues/load-1
              (fn [{:keys [db]} [_ leagues]]
                {:db (assoc db :leagues (filter (comp whitelist :id) leagues))}))
```

The exact same events can be created with chains like this:

```clojure
(reg-chain :leagues/load

           (fn [_ _]
             {:http-xhrio   {:uri "http://api.football-data.org/v1/competitions/?season=2017"}})

           (fn [{:keys [db]} [_ leagues]]
             {:db (assoc db :leagues (filter (comp whitelist :id) leagues))}))
```

## Navigation and links


Trigger a browser navigation to the URL `/league/3/table`:

```clojure
(reg-event-fx :leagues/select
              (fn [_ [league-id]]
                {:navigate-to [:league :id league-id :tab :table]}))
```

Generate a good old href from plain data:

```clojure
[:a.nav-link {:href (k/path-for [:league :id 3 :tab :fixtures])} "Latest results"]
```

Render different views depending on what URL you're on

```clojure
[k/switch-route (fn [route-data] (:handler route-data))
    :league [league/league-dispatch]
    :live   [live/live]
    nil     [:div "Loading..."]]
```

## And we're done!

That should cover most of the API for kee-frame. There are of course more details, head over to the
[project page](https://github.com/ingesolvoll/kee-frame) to learn more!
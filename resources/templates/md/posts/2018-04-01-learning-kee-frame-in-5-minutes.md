{:title "Learning kee-frame in 5 minutes"
 :layout :post
 :tags  ["clojurescript" "re-frame" "kee-frame"]}

You are here, so you probably heard about the [kee-frame framework](https://github.com/ingesolvoll/kee-frame). It's a small framework
built on re-frame, and should be easy to learn. Let's see if that's true!

## Installation

If you're starting a brand new project, I would recommend using a lein template, like this:

```clojure
lein new re-frame <your-project-name>
```

Once you have a working re-frame project, just add kee-frame as a dependency to your project:

```clojure
[kee-frame "0.1.8"]
```

Now run `lein figwheel` and we're good to go.

## Startup

The `start!` function hooks everything up, including **routing, spec-validation, debug logging, initial db and rendering** .

```clojure
(def routes ["" {"/"                       :live
                 ["/league/" :id "/" :tab] :league}])

(def initial-db {:drawer-open?  false
                 :leagues       nil})

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

The next controller starts and stops a polling loop when we enter and leave the live page. The stop is triggered by `:params`
returning nil.

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
                                :on-success [:leagues/received]}}))

(reg-event-fx :leagues/received
              (fn [{:keys [db]} [_ leagues]]
                {:db (assoc db :leagues (filter (comp whitelist :id) leagues))}))
```

With event chains, they can be expressed like this:

```clojure
(reg-chain :leagues/load

           (fn [_ _]
             {:http-xhrio   {:uri "http://api.football-data.org/v1/competitions/?season=2017"}})

           (fn [{:keys [db]} [_ leagues]]
             {:db (assoc db :leagues (filter (comp whitelist :id) leagues))}))
```


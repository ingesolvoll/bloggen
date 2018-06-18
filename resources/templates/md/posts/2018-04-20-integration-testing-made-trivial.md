{:title "Integration testing made trivial"
 :layout :post
 :draft? true
 :tags  ["clojure" "testing"]}

## Chromedriver
We'll be using chromedriver for this exercise. [Download the driver for your OS](https://sites.google.com/a/chromium.org/chromedriver/)
and put it on your PATH.

## Etaoin
Include the dependency in your `project.clj`:

`[etaoin "0.2.8"]`

We need an empty namespace suited for etaoin REPL development:

```clojure
(ns kee-frame-sample.integration-test
  (:require [clojure.test :refer :all]
            [etaoin.api :as et]))

(deftest first-test
  (is (= 1 2)))
```

## Launching the REPL
Let's start REPLing. `lein repl` should get you going. Once that has launched, go to our test namespace like this:

`(in-ns 'kee-frame-sample.integration-test)`

You can verify that everything works by running your example failing test like this:

`(run-tests)`
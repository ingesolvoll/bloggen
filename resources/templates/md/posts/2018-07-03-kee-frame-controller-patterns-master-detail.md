{:title "Kee-frame controller patterns: master/detail"
 :layout :post
 :tags  ["clojurescript" "re-frame" "kee-frame"]}

## The problem
Most non-trivial single page apps needs to deal with this sooner or later:

* Load a list of items
* Load a list of items AND load details for a single items

##  Low coupling, high cohesion
The AND is the key here. You can of course hand-wire the logic for these 2 partially overlapping cases, but reality comes at
you fast. A couple more cases and nested if statements later, every new change request is followed by a couple of bug reports
on existing functionality that stopped working.


```clojure
```
{:title "Kee-frame patterns: master/detail"
 :layout :post
 :tags  ["clojurescript" "re-frame" "kee-frame"]}

## The problem
Most non-trivial single page apps deals with the problem of master/detail sooner or later:

* Load a list of items
* Load a list of items AND load details for a single item

##  Low coupling, high cohesion
You can of course hand-wire the logic for these 2 partially overlapping cases, but reality comes at
you fast. A couple of nested if statements later, every new feature addition is followed by bug reports on existing functionality.
There are several reasons for this to grow out of hand. Here are a few:

### Client-side caching
You may want to cache the list of items so you don't load it again when another single item is loaded. You also might want to
update the list after editing a single item. Maybe you deleted it, maybe you changed the name showed in the list.


```clojure
```
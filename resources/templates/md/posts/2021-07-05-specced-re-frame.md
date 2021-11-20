{:title "A la carte specs for your re-frame subs and events"
 :layout :post
 :tags  ["re-frame" "spec" "clojurescript"]}

## The problem
Re-frame subscriptions and events are like very loosely defined function calls. There's a contract there
between the sender and the receiver, but it's implicit. The sender makes sure there are 3 items in the vector.
The receiver destructures naively, expecting exactly 3 items.

```clojure
(rf/reg-event-db ::do-something 
                 (fn [db [_ first second {:keys [some-key]}]]
                  ....))

(rf/dispatch [::do-something 1 2 {:some-key 3}])
```

Keeping these in sync requires continuous effort, and it's easy to miss a case or two, even in less complex
applications. Because the errors often involve incompatible use of core clojure language features, like vector and map
destructuring on things that cannot be destructured, they are quite often incomprehensible and hard to
trace. 

## The solution (for me)

Lately, I've been searching for some gradual typing to ease the pain. Most of the available tools based on spec
or other schema libraries are capable of solving the above problem. But only one of them check all my points:

- Instrument function calls at dev time
- Check both function args and return values
- Support anonymous functions (we write a LOT of those in re-frame apps and extracting them into named functions isn't an attractive alternative)
- Compact and inline syntax

## speced.def
[speced.def][speced] is to me a pretty optimal solution for re-frame vectors. You can annotate
as much or little as you want, which is really important to avoid too much boilerplate. Here's the same example
from above:

```clojure
(s/def ::int int?)
(s/def ::valid-db #(:some-crucial-key %))

(rf/reg-event-db ::do-something 
                 (d/fn ^::valid-db [db [_ ^::int   first 
                                          ^number? second
                                                   {:keys [^:int some-key]}]]
                  ....))
```

The first thing you should notice is that I replaced clojure's own `fn` with a version that checks its own
args and return value. The syntax is still completely standard, and should be easily adaptable in your IDE.
I'm using Cursive, and it can be told to treat the `d/fn` as if it was `clojure.core/fn`

As you can see, I annotated the first 2 args with specs, and we're even able to verify the type of the
one map key we are interested in, in a pretty compact way. I also add a validity check for the updated app db returned 
from the event.

As I mentioned, other spec libraries can do this too, but they require you to write a spec for the entire
args vector, which to me is way too much boilerplate to be maintainable in the long term. And it's no fun.

With this system in place, and if you are lucky enough to have [expound][expound] running, you
are entering a totally different world of error messages and early failures for the many mistakes you are bound
to make.

[speced]: https://github.com/nedap/speced.def
[expound]: https://github.com/bhb/expound
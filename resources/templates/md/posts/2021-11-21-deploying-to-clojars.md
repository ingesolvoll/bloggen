{:title "Deploying to clojars"
 :layout :post
 :tags  []}

## Deploying to clojars
This is just a very short post to document my port of 
[this excellent blog solution](https://slipset.github.io/posts/deploying-to-clojars). Before you continue reading,
you should read that other one first as I won't repeat all the relevant info here. In particular, you need to set up
a Circle CI context containing your credentials.

As the original post says, if deployment depends on a number of manual steps, it quickly becomes a barrier to getting
new versions out. The solution outlined here automates everything, so the only thing you need to do is to create a
new release tag on github that looks like `Release-1.2.3`.

## Steps
The original post contains most of the info you need. This post focuses on the steps specific to tools.deps. The build
tooling around tools.deps is becoming really streamlined and makes it quite easy to achieve the same setup. It even
saves you from having to make the project-version runtime configurable, as the version is easily available in your build
script.

### deps.edn
Here's your minimal project file.
```clojure
{:paths   ["src"]
 :aliases {:build {:deps       {io.github.seancorfield/build-clj
                                {:git/tag "v0.5.0" :git/sha "2ceb95a"}}
                   :ns-default build}}
 :deps    {org.clojure/clojure {:mvn/version "1.10.1"}}}
```

### build.clj

This is the major difference from the leiningen approach. Since the build script is just clojure, you can access 
the tag/version directly without any further trickery.

```clojure
(ns build
  (:require
   [clojure.string :as str]
   [org.corfield.build :as bb]))

(def lib 'your/lib)

(def release-marker "Release-")

(defn extract-version [tag]
  (str/replace-first tag release-marker ""))

(defn maybe-deploy [opts]
  (if-let [tag (System/getenv "CIRCLE_TAG")]
    (do
      (println "Found tag " tag)
      (if (re-find (re-pattern release-marker) tag)
        (do
          (println "Deploying to clojars...")
          (-> opts
              (assoc :lib lib :version (extract-version tag))
              (bb/jar)
              (bb/deploy)))
        (do
          (println "Tag is not a release tag, skipping deploy")
          opts)))
    (do
      (println "No tag found, skipping deploy")
      opts)))
```

## Running it

From here, you just run `clojure -T:build maybe-deploy`. Insert that command in your config.yml. [Here's a working example YAML](https://github.com/ingesolvoll/re-statecharts/blob/main/.circleci/config.yml)

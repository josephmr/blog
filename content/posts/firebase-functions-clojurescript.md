+++
title = "Firebase Functions in ClojureScript"
author = ["Joseph Rollins"]
description = "Use ClojureScript to write your Firebase Functions."
date = 2021-04-12
categories = ["clojurescript"]
draft = false
+++

As I started on a less trivial project to learn ClojureScript more via
re-frame I reached for Google Cloud Firestore as a simple database. I
generally use Firestore when prototyping as I am familiar with it and it
requires minimal setup.

It is common when using Firestore to leverage Firebase Functions to control
some writes to the database. For example, to prevent cheating I used a
Firebase Function to validate moves being played in a tic-tac-toe game.

Instead of writing the functions in Javascript or Typescript, it is possible
to write them in ClojureScript and compile to Javascript pretty easily.


## Create a shadow-cljs Project {#create-a-shadow-cljs-project}

To get started we will initialize a new shadow cljs project. shadow cljs
(shadow-cljs) is responsible for compiling our clojurescript and leveraging
existing Javascript libraries (e.g. firebase-functions).

```bash
npx create-cljs-project echo
```

This should generate the following project for us:

```sh
➜ src tree -I node_modules echo
echo
├── package.json
├── package-lock.json
├── shadow-cljs.edn
└── src
    ├── main
    └── test
```


## Initialize Firebase Functions {#initialize-firebase-functions}

Now we need to initialize Firebase in the directory and configure it for writing functions.

```sh
firebase init
# choose Functions
# choose a GCP project
# choose Javascript
# decline EsLint
```

This will create a new `functions` directory with a `package.json` file for
our functions Javascript dependencies and a basic `index.js` file which we
won't be using.


## Write our ClojureScript {#write-our-clojurescript}

Let's write a basic function in `src/main/echo/fn.cljs`. This will be the
function we deploy to Firebase.

```clojure
(ns echo.fn
  (:require ["firebase-functions" :as functions]
            ["firebase-admin" :as admin]))

(defonce init (.initializeApp admin))

(defn echo
  "Echo the passed in query parameters merged with the current time"
  [req res]
  (let [query (js->clj (.-query req) :keywordize-keys true)]
    (.json res (clj->js {:time (.toString (js/Date.))
                         :query query}))))

(def exports
  "Configure exports that Firebase expects (i.e. module.exports)"
  #js {:echo (.onRequest functions/https echo)})
```

First, we import the `firebase-functions` and `firebase-admin` Javascript
packages. These dependencies, and all of our other Javascript dependencies
are declared in `functions/package.json`.

Next, we have a relatively simple `echo` function. As this functions is
using the Firebase `onRequest` function style we expect to receive a
`request` and `response` object as parameters.

We decode the query parameters from the request, converting into
ClojureScript data structures via `js->clj`. Then, we call `res.json` to
send back the current time and the passed query parameters. The inverse
conversion, `clj->js` is used to return Javascript data structures back.

Finally, we expose the `echo` function in the module exports.


## Compile our ClojureScript {#compile-our-clojurescript}

Now we just need to configure Shadow CLJS to correctly compile our
ClojureScript. Edit the `shadow-cljs.edn` file to match the following.

```clojure
{:source-paths
 ["src/main"]

 :dependencies
 []

 :builds
 {:fn {:target :node-library
       :js-options {:js-package-dirs ["functions/node_modules"]}
       :compiler-options {:infer-externs :auto}
       :output-to "functions/index.js"
       :exports-var echo.fn/exports}}}
```

We add a new build `:fn` which uses the `functions/node_modules` for
Javascript dependencies, outputs the compiled Javascript to
`functions/index.js` and identify the exported code via its symbol.

Finally, we can compile our code by running the following.

```sh
npx shadow-cljs compile :fn
```

This will output our compiled code to `functions/index.js` ready to be tested
using the Firebase emulators or deployed.


## References {#references}

[Full Code on Github](https://github.com/josephmr/clojurescript-firebase-functions)

[Helpful example from Jacob O'Bryant](https://github.com/jacobobryant/mystery-cows)

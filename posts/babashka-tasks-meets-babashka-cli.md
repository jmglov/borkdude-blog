In a previous blog post
[(link)](https://blog.michielborkent.nl/babashka-cli.html) I introduced
[babashka CLI](https://github.com/babashka/cli). It offers you similar benefits
as `clojure -X` but with a more Unixy command line interface.

In version `0.9.160` of babashka, babashka CLI was integrated.  It is available
as a built-in library so you don't need to declare it anymore in `:deps` in
`bb.edn` unless you want to use a newer version than the built-in one.

## -x

For invoking functions from the command line, you can use the new `-x` flag (a pun to Clojure's `-X` of course!):

``` clojure
bb -x clojure.core/identity --hello there
{:hello "there"}
```

What we see in the above snippet is
that a map `{:hello "there"}` is constructed by babashka CLI and then fed to the
`identity` function. After that the result is printed to the console.

What if we want to influence how things are parsed by babashka CLI and provide
some defaults? This can be done using metadata. Let's create a `bb.edn` and make
a file available on the classpath:

`bb.edn`:
``` clojure
{:paths ["."]}
```

`tasks.clj`:
``` clojure
(ns tasks
  {:org.babashka/cli {:exec-args {:ns-data 1}}})

(defn my-function
  {:org.babashka/cli {:exec-args {:fn-data 1}
                      :coerce {:num [:int]}
                      :alias {:n :num}}}
  [m]
  m)
```

Now let's invoke:

``` clojure
$ bb -x tasks/my-function -n 1 2
{:ns-data 1, :fn-data 1, :num [1 2]}
```

As you can see, the namespace options are merged with the function
options. Defaults can be provided with `:exec-args`, like you're used to from
the clojure CLI.

## Tasks

What about task integration? Let's adapt our `bb.edn`:

``` clojure
{:paths ["."]
 :tasks {doit {:task (let [x (exec 'tasks/my-function)]
                       (prn :x x))
               :exec-args {:task-data 1234}}
         }}
```

and invoke the task:

``` clojure
$ bb doit --cli-option :yeah -n 1 2 3
:x {:ns-data 1, :fn-data 1, :task-data 1234, :cli-option :yeah, :num [1 2 3]}
```

As you can see it works similar to `-x`, but you can provide another set of
defaults on the task level with `:exec-args`. Executing a function through
babashka CLI is done using the `babashka.task/exec` function, available by default
in tasks.

<!-- A hack to parse command line arguments in babashka and then forward them to Clojure: -->

<!-- ``` clojure -->
<!-- {:paths ["."] -->
<!--  :tasks -->
<!--  {:requires ([babashka.cli :as cli]) -->

<!--   doit {:task -->
<!--         (do -->
<!--           (defn parse {:org.babashka/cli {:coerce {:number [:int]} -->
<!--                                           :alias {:n :number}}} -->
<!--             [m] m) -->
<!--           (clojure "-X" "clojure.core/prn" -->
<!--                    (exec parse))) -->
<!--         :exec-args {:task-data 1234}} -->
<!--   }} -->
<!-- ``` -->

<!-- ``` clojure -->
<!-- $ bb doit -n 1 2 3 -->
<!-- {:task-data 1234, :number [1 2 3]} -->
<!-- ``` -->

<!-- although you can do the above manually using `(babashka.cli/parse-opts ...)` as -->
<!-- well or by using babashka CLI directly with the [clojure CLI](https://github.com/babashka/cli#clojure-cli). -->

Hope you will enjoy this!

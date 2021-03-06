# clojure-repl

Very simple project skeleton for running the REPL and doing experiments.

## Adding dependency 

Latest stable version: 
[![Clojars Project](https://img.shields.io/clojars/v/curiousprogrammer/clojure-repl.svg)](https://clojars.org/curiousprogrammer/clojure-repl)

Add `clojure-repl` as a dependency to your profiles.clj or project.clj.
Ideally, into the `:repl` profile to not interfere it with other tasks:

```
 :repl {:dependencies [[curiousprogrammer/clojure-repl "x.y.z"]]}}
```

## Libraries

### [criterium](https://github.com/hugoduncan/criterium)

Can be used to do quick micro-benchmarks.
Check https://github.com/hugoduncan/criterium#usage.

### [alembic](https://github.com/pallet/alembic)

Can be used to add a new dependency to classpath dynamically without restarting the REPL.
It's used internaly by [nrepl-refactor](https://github.com/clojure-emacs/refactor-nrepl/blob/a425a8103413fe91f56907857c2043c32b3630a2/src/refactor_nrepl/artifacts.clj#L111).

You can add dependency like this:

```
(require '[alembic.still :as a])

(a/distill '[org.jasypt/jasypt "1.9.2"])
;;=> Loaded dependencies:
;;=> [[org.jasypt/jasypt "1.9.2"]]
;;=> nil
```

Can also be used to invoke leiningen tasks programatically: https://github.com/pallet/alembic#invoking-leiningen-tasks

```
(require '[alembic.still :as a])

(a/lein deps :tree)
;;=> lots of output...
```

### clj-memory-meter

[clj-memory-meter** is a nice tool for measuring the memory occupied by some Java/Clojure object.

Use it:

```
(require '[clj-memory-meter.core :as mm])
(mm/measure [1 2 3 4 5])
nil
384 B
```

**Info**:

* [clj-memory meter – measure the memory used by arbitrary objects](https://groups.google.com/forum/#!topic/clojure/MIdLIvo07Vw)
* http://clojure-goes-fast.com/blog/introspection-tool-object-memory-meter/
  *  It wraps JAMM, on top of which it provides the ability to load the JVM agent at runtime (so you don't need to start the program with -javaagent parameter) 
  * measure is the only function you would use. It walks the object and its components, calculates the total memory occupancy, and returns a human-readable result. You can call it on any Java or Clojure object
  * You can provide :shallow true as a parameter to do only a shallow analysis of the object's memory usage. It counts the object header plus the space for object fields, without following the references.
  * You can pass :debug true to measure to print the object layout tree with sizes for each part. Or you can pass :debug <number> to limit the nesting level being printed:
* https://github.com/clojure-goes-fast/clj-memory-meter
  * This library is a thin wrapper around Java Agent for Memory Managements. It allows to inspect at runtime how much memory an object occupies together with all its child fields.
  * uses https://github.com/jbellis/jamm


### JOL (Java Object Layout) - UPDATE: prefer clj-memory-meter

Handy library and command line tool for analyzing java objects' layout and estimating their size.
Check http://openjdk.java.net/projects/code-tools/jol/.

Check also examples: http://hg.openjdk.java.net/code-tools/jol/file/tip/jol-samples/src/main/java/org/openjdk/jol/samples/

The basic example: http://hg.openjdk.java.net/code-tools/jol/file/018c0e12f70f/jol-samples/src/main/java/org/openjdk/jol/samples/JOLSample_01_Basic.java

```
(import '(org.openjdk.jol.info ClassLayout))
(.toPrintable (ClassLayout/parseInstance []))
;;=>
clojure.lang.PersistentVector object internals:
 OFFSET  SIZE                                 TYPE DESCRIPTION                               VALUE
      0     4                                      (object header)                           21 00 00 00 (00100001 00000000 00000000 00000000) (33)
      4     4                                      (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4                                      (object header)                           c7 df 00 f8 (11000111 11011111 00000000 11111000) (-134160441)
     12     4                                  int APersistentVector._hash                   -1
     16     4                                  int APersistentVector._hasheq                 -2017569654
     20     4                                  int PersistentVector.cnt                      0
     24     4                                  int PersistentVector.shift                    5
     28     4   clojure.lang.PersistentVector.Node PersistentVector.root                     (object)
     32     4                   java.lang.Object[] PersistentVector.tail                     []
     36     4          clojure.lang.IPersistentMap PersistentVector._meta                    null
Instance size: 40 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

**Note**: *you may want to clone source repository to get latest version*:

```
hg clone http://hg.openjdk.java.net/code-tools/jol/ jol
cd jol
mvn clean install
```

Then update version in project.clj.


#### Using GraphLayout

`GraphLayout` is one of more useful classes which can be used for getting total object footprint.
Check [ObjectFootprint.java source code](http://hg.openjdk.java.net/code-tools/jol/file/018c0e12f70f/jol-cli/src/main/java/org/openjdk/jol/operations/ObjectFootprint.java).

One problem with this class is that `parseInstance` method accepts varargs,
which means it's harder to use from Clojure.
You can use it like this:

```
(import '(org.openjdk.jol.info GraphLayout))

(.toFootprint (GraphLayout/parseInstance (doto (object-array 1) (aset 0 [1 2 3]))))
;;=>
clojure.lang.PersistentVector@4999f4e6d footprint:
     COUNT       AVG       SUM   DESCRIPTION
         2        88       176   [Ljava.lang.Object;
         1        40        40   clojure.lang.PersistentVector
         1        24        24   clojure.lang.PersistentVector$Node
         3        24        72   java.lang.Long
         1        16        16   java.util.concurrent.atomic.AtomicReference
         8                 328   (total)
```

Check https://stackoverflow.com/questions/11702184/how-to-handle-java-variable-length-arguments-in-clojure
for more details.

## API

### Java

List all public java methods (including inherited ones) in given java class:

```clojure
(require '[clojure-repl.java :as j])
(j/jmethods java.util.Date)
```

Show inheritance tree:

```
(require '[clojure-repl.java :as j])

(j/ancestors clojure.lang.PersistentArrayMap)
* <clojure.lang.PersistentArrayMap>
    * <clojure.lang.APersistentMap>
        * <clojure.lang.AFn>
            * clojure.lang.IFn
                * java.lang.Runnable
                * java.util.concurrent.Callable
            * <java.lang.Object>
        * clojure.lang.IHashEq
        * clojure.lang.IPersistentMap
            * clojure.lang.Associative
                * clojure.lang.ILookup
                * clojure.lang.IPersistentCollection
                    * clojure.lang.Seqable
            * clojure.lang.Counted
            * java.lang.Iterable
        * clojure.lang.MapEquivalence
        * java.io.Serializable
        * java.lang.Iterable
        * java.util.Map
    * clojure.lang.IEditableCollection
    * clojure.lang.IKVReduce
    * clojure.lang.IMapIterable
    * clojure.lang.IObj
        * clojure.lang.IMeta

;; or use more verbose
(j/print-tree (j/inheritance-tree clojure.lang.PersistentArrayMap))

```

Note: the implementation code for inheritance trees was adapted from Stuart Sierra's talk [Learning Clojure: Next Steps](https://www.youtube.com/watch?v=pbodL96HM28).

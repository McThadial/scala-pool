# scala-pool

*scala-pool* is a Scala library for object pooling. The library provides an API and different pool
 implementations that allow:

  - blocking/non-blocking object acquisition
  - object invalidation
  - capping the number of pooled objects
  - creating new objects lazily, as needed
  - health checking
  - time-based pool eviction (idle instances)
  - GC-based pool eviction (soft and weak references)
  - efficient thread-safety

* * *

[![Build Status](https://img.shields.io/travis/andrebeat/scala-pool/master.svg)](https://travis-ci.org/andrebeat/scala-pool)
[![Coverage](https://img.shields.io/coveralls/andrebeat/scala-pool/master.svg)](https://coveralls.io/github/andrebeat/scala-pool)
[![License](https://img.shields.io/dub/l/vibe-d.svg)](https://raw.githubusercontent.com/andrebeat/scala-pool/master/LICENSE)

[Scaladoc](https://andrebeat.github.io/scala-pool/latest/api/index.html#io.github.andrebeat.pool.package)

## Installation

scala-pool's latest release is `0.1.0` and is built against Scala `2.11.7`.

To use it in an existing SBT project, add the following dependency to your `build.sbt`:

```scala
libraryDependencies += "io.github.andrebeat" %% "scala-pool" % "0.1.0"
```

The latest snapshot version is also available:

```scala
libraryDependencies += "io.github.andrebeat" %% "scala-pool" % "0.2.0-SNAPSHOT"
```

It might be necessary to add the Sonatype OSS Snapshot resolver:

```scala
resolvers += Resolver.sonatypeRepo("snapshots")
```

Currently, the library has no external dependencies apart from the Java and Scala standard
libraries.

## Usage

The basic usage of the pool is shown below:

```scala
import io.github.andrebeat.pool._

// Creating a `Pool[Object]` with a capacity of 2 instances
val pool = Pool(2, () => new Object)

// Acquiring a lease on an object from the pool (blocking if none available)
val lease = pool.acquire()

// Using the lease
lease.use { o =>
  println(o)
}

// The object is returned to the pool at this point
```

All of the different pool features are exposed in the `Pool` companion object `apply` method:

```scala
Pool(
  capacity: Int,                // the maximum capacity of the pool
  factory: () => A,             // the function used to create new objects in the pool
  referenceType: ReferenceType, // the reference type of objects in the pool
  maxIdleTime: Duration,        // the maximum amount of the time that objects are allowed
                                //   to idle in the pool before being evicted
  reset: A => Unit,             // the function used to reset objects in the pool
                                //   (called when leasing an object from the pool)
  dispose: A => Unit,           // the function used to destroy an object from the pool
  healthCheck: A => Boolean)    // the predicate used to test whether an object is
                                //   healthy and should be used, or destroyed otherwise
```

It is also possible to get a value from a lease and release it (or invalidate) manually.

```scala
import io.github.andrebeat.pool._

// Creating a `Pool[Object]` with a capacity of 2 instances
val pool = Pool(2, () => new Object)

// Getting the value from the lease
val obj = lease.get()

// There are currently no objects on the pool
pool.size
// res0: Int = 0

// But its capacity is 2 (objects are created lazily)
pool.capacity
// res1: Int = 2

// There's 1 live object
pool.live
// res2: Int = 1

// And its currently leased
pool.leased
// res3: Int = 1

// Releasing our lease back to the pool
lease.release

// Its now in the pool waiting to be reused
pool.size
// res4: Int = 1
```

The API is documented in depth in the [Scaladoc](https://andrebeat.github.io/scala-pool/latest/api/index.html#io.github.andrebeat.pool.package).

## License

scala-pool is licensed under the [MIT](http://opensource.org/licenses/MIT) license. See `LICENSE`
for details.

## How can a Map be sorted in Java?

Using TreeMap.

## Main difference in java 17 and java 8

Java 8:
- Lambda expressions
- Functional interfaces
- Streams API
- Optional class
- Default & static methods in interfaces

Java 17:
- Records → concise data classes
```
record User(String name, int age) {}
```
- Sealed classes -> restrict inheritance
- Pattern matching for instanceof
```
if (obj instanceof String s) { }
```
- Text blocks
```
String text = """Hello, 
World!""";
```
- Switch expressions
```
switch (x) { case 1 -> "One"; case 2 -> "Two"; default -> "Other"; }
```
- Performance Improvements
  - Better JVM optimizations
  - New garbage collectors: G1 improved (default), ZGC (low-latency), Shenandoah
  - Java 17 is significantly faster and more memory-efficient in real systems.
- Security & Encapsulation
  - Java 8: Weak module boundaries
  - Java 17: Strong encapsulation via JPMS (Java Platform Module System, introduced in Java 9)
  - Internal APIs are no longer easily accessible (safer but may break old code)
- Module System
  - Not present in Java 8
  - Introduced in Java 9, fully part of Java 17

- API Enhancements
  - New methods in String, Optional, Stream
  - HttpClient (modern replacement for HttpURLConnection)
  - Improved collectors
  - Files API improvements

## Runnable vs Callable

The main differences between `Runnable` and `Callable` are:

- **Return Value**: `Runnable`'s `run()` method returns `void` (no result), whereas `Callable`'s `call()` method returns a generic value `V`.
- **Exception Handling**: `Runnable` cannot throw checked exceptions (no `throws` clause in `run()`). In contrast, `Callable` can throw checked exceptions because its `call()` method has a `throws Exception` clause.
- **Introduction**: `Runnable` was introduced in Java 1.0, while `Callable` was introduced in Java 1.5.

**Where they are used:**
- Use **Runnable** for executing concurrent background tasks where a result or status isn't needed (e.g., background polling, logging).
- Use **Callable** for concurrent tasks where you need the result returned, or when the task can throw checked exceptions. It's commonly used with `ExecutorService`.

**Basic Implementation:**
> ```java
> // Runnable implementation
> class MyRunnable implements Runnable {
>     @Override
>     public void run() {
>         System.out.println("Runnable is running");
>     }
> }
> 
> // Runnable Usage
> Thread thread = new Thread(new MyRunnable());
> thread.start();
> ```
> 
> ```java
> // Callable implementation
> import java.util.concurrent.Callable;
> import java.util.concurrent.ExecutorService;
> import java.util.concurrent.Executors;
> import java.util.concurrent.Future;
> 
> class MyCallable implements Callable<String> {
>     @Override
>     public String call() throws Exception {
>         boolean errorOccurred = true; // Simulate some condition
>         if (errorOccurred) {
>             throw new Exception("An error occurred during execution");
>         }
>         return "Callable result";
>     }
> }
> 
> // Callable Usage
> ExecutorService executor = Executors.newSingleThreadExecutor();
> Future<String> future = executor.submit(new MyCallable());
> 
> try {
>     String result = future.get(); // blocks until result is available
>     System.out.println(result);
> } catch (java.util.concurrent.ExecutionException e) {
>     // Exceptions thrown inside Callable are wrapped in ExecutionException
>     System.out.println("Callable threw an exception: " + e.getCause().getMessage());
> } catch (InterruptedException e) {
>     e.printStackTrace();
> } finally {
>     executor.shutdown();
> }
> ```

## How is strip better than trim
### trim() (Java 8)
- Removes only ASCII whitespace
- Works based on characters ≤ U+0020 (space, tab, newline, etc.)
- Fails for many Unicode spaces

### strip() (Java 11+, available in Java 17)
- Uses Unicode-aware whitespace detection
- Based on Character.isWhitespace()

## == vs equals()
- `==` is an **operator** that compares memory references (i.e., whether two variables point to the exact same object in memory).
- `equals()` is a **method** that compares the actual logical content (value) of the objects, assuming the class overrides the method (like `String` does).
```java
String s1 = new String("Java");
String s2 = new String("Java");

System.out.println(s1 == s2);      // false (different memory locations)
System.out.println(s1.equals(s2)); // true (same logical text)
```

## How HashMap works internally
A `HashMap` stores Key-Value pairs using an array of "buckets" (Nodes).
1. **Hashing**: When you call `put(key, value)`, it calculates the `hashCode()` of the key to generate an integer.
2. **Index Calculation**: It uses this hash combined with the array size to pick a bucket index.
3. **Storing**: It places a Node containing `[Hash, Key, Value, Next Node Reference]` at that index.

## Collision handling (LinkedList vs Tree in Java 8+)
A **collision** occurs when two distinct keys yield the exact same bucket index.
- **Before Java 8**: Collisions were managed by creating a **LinkedList** at that index. Searching a long list had slow performance: `O(n)`.
- **Java 8+**: If a single bucket's LinkedList grows beyond **8 elements** (and the overall map holds at least 64 items), Java automatically upgrades it into a **Balanced Red-Black Tree**, drastically reducing search time to `O(log n)`.

## Load factor reasoning (0.75)
The **load factor** controls when the `HashMap` automatically resizes (usually doubling) its bucket array.
- **Why exactly 0.75?** It is the optimal mathematical sweet spot.
  - A **higher** load factor (e.g., 1.0) saves memory but dramatically increases collisions, slowing down reads/writes.
  - A **lower** load factor (e.g., 0.5) eliminates collisions but wastes massive amounts of memory due to frequent array resizing.

## Java Memory Model (JMM)
The JMM defines how threads interact through memory and dictates rules about visibility, ordering, and synchronization.
**Core Concept**: Threads do not always write directly to the "Main Memory" (RAM). Mostly, they read/write to their own hidden high-speed **CPU Caches** (Local Memory), which can cause other concurrent threads to temporarily read stale/old values!

```mermaid
graph TD
    Main[Main Memory RAM <br/> Shared Variables]
    
    subgraph Thread 1 Environment
    T1Mem[Thread 1 Local Memory <br/> CPU Cache] 
    T1[Thread 1]
    end
    
    subgraph Thread 2 Environment
    T2Mem[Thread 2 Local Memory <br/> CPU Cache]
    T2[Thread 2]
    end

    Main <-->|Read / Flush Updates| T1Mem
    T1Mem <-->|Variables| T1
    
    Main <-->|Read / Flush Updates| T2Mem
    T2Mem <-->|Variables| T2
```
> *Note: The `volatile` Java keyword forces a thread to bypass its Local Memory and read/write directly to Main Memory, ensuring other threads immediately see the updated value.*

## Thread safety & concurrency
**Thread Safety** means your code functions perfectly even when multiple threads execute it simultaneously without data corruption.
- **Race conditions**: Happen when two threads try to update the same variable simultaneously, blindly overwriting each other's changes.
- **How to prevent it**:
  1. **`synchronized` block / method**: Locks the block so only one thread can execute it at a time.
  2. **`volatile` keyword**: Solves visibility issues (ensures threads don't read "stale" values from their cache).
  3. **Concurrent Collections**: Use classes from `java.util.concurrent` (like `ConcurrentHashMap` or `AtomicInteger`) instead of standard collections, as they handle locking automatically and with far better performance than standard `synchronized` blocks.

## synchronized(className.class) vs synchronized(objectName.class)
- `synchronized(className.class)`: Locks the **Class Object** (Singleton lock). Only one thread can execute *any* synchronized method/block on that class at a time, regardless of how many instances exist.
- `synchronized(objectName.class)`: Locks the **Specific Instance** (Object lock). Only one thread can execute synchronized code on *that specific object* at a time. Other instances of the same class can be processed by other threads simultaneously.

## synchronized vs ReentrantLock vs ReadWriteLock

### 1. `synchronized`
- **What it is**: The fundamental base Java keyword that handles thread interference. When a thread enters a synchronized block on an object, no other thread can enter *any* synchronized block on that exact same object.
- **Inter-thread Communication (`wait()` and `notify()`)**:
  - `synchronized` is designed to work seamlessly with `wait()` and `notify()`.
  - **`wait()`**: Forces the thread to immediately release the lock it currently holds and go to "sleep" until another thread wakes it up.
  - **`notify()`**: Wakes up a *single* sleeping thread that is actively waiting on this object's lock (`notifyAll()` wakes them all). 
  > *Crucial rule: You can only call `wait()` or `notify()` securely from firmly **inside** a `synchronized` block!*
- **Limitations**: You cannot interrupt a waiting thread, it blocks forever until the lock is naturally released, and it does not guarantee fairness (a waiting thread could starve indefinitely).
- **When to use**: Simple, basic synchronization scenarios where massive performance control is not desperately needed.

### 2. `ReentrantLock`
- **What it is**: An advanced class from `java.util.concurrent.locks`. "Reentrant" literally means that if a thread already holds the lock, that identical thread can safely request the exact same lock *again* without deadlocking itself (it simply increments an internal counter).
- **Advanced Control Features**: 
  - **`tryLock()`**: You can dynamically attempt to grab the lock but give up immediately or after a set timeout if someone else holds it (e.g., `if (lock.tryLock(5, TimeUnit.SECONDS))`). This brilliantly solves infinite blocking.
  - **Fairness Policy**: You can pass `true` to the constructor (`new ReentrantLock(true)`) to aggressively guarantee that the longest-waiting thread gets the lock next.
- **When to use**: When you desperately need advanced timeouts, non-blocking lock checks, or strict "fairness" queue protocols.

### 3. `ReadWriteLock`
- **What it is**: It physically manages two discrete locks internally simultaneously: a **Read Lock** and a **Write Lock**.
- **How it works**: 
  - **Read Lock**: Multiple threads can hold this lock *simultaneously* (as long as no thread holds the Write lock). 
  - **Write Lock**: A completely exclusive lock. If a thread grabs the Write lock, no other thread is allowed to read *or* write.
- **When to use**: Highly read-intensive applications (e.g., a cache layer that realistically reads 1,000 times for every 1 write). It drastically skyrocketed performance by letting thousands of threads read concurrently, completely eliminating the reading bottleneck caused by standard `synchronized` blocks.

## Fail-fast vs Fail-safe Iterators
- **Fail-fast**: Used by standard collections (`ArrayList`, `HashMap`). If you modify a collection while iterating through it, the iterator instantly throws a `ConcurrentModificationException`. It works directly on the memory of the original collection.
- **Fail-safe**: Used by concurrent collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`). These will **not** crash if the collection changes during iteration. They usually work by iterating over a snapshot/clone of the collection at the time the iterator was created.

## String Immutability and Multithreading
- **Why are they immutable?**: To enable the **String Pool** (saving highly valuable memory by making identical strings point to the same object) and for **Security** (safely storing SQL connections or passwords).
- **Multithreading Benefits**: Because a String can physically *never change* its value after creation, it is **100% thread-safe**. You can pass a String safely across hundreds of threads without writing a single `synchronized` block. If a thread attempts to alter the String, it is forced to create a brand new String object instead.

## ForkJoinPool and Work-Stealing
- **How it works**: Designed for multi-processor efficiency using "Divide and Conquer". It breaks a massive task down into tiny subtasks (**Fork**), runs them across available CPU cores, and combines the results (**Join**).
- **Work-Stealing**: If a thread finishes its subtasks early, it doesn't sit idle. It "steals" pending tasks from the queues of other busy threads, ensuring maximum CPU utilization.
- **When to use**: CPU-intensive algorithms that can be recursively split (e.g., Image processing, sorting enormous arrays). *Do not use it for I/O bound tasks like database or API blocking calls.*

## Java Streams (Filter, Map, Reduce & Parallel)
- **Functional Processing**: Instead of manually writing `for` loops with complex `if` statements, Streams allow you to declaratively state *what* you want to achieve (`filter()` bad data, `map()` it to a new format, `reduce()` it to a sum).
- **Lazy Evaluation**: Stream operations are aggressively ignored until a **Terminal Operation** is invoked. If you filter a billion items but never trigger a terminal operation, absolutely zero execution happens.
- **Sequential vs Parallel**: By default, streams are sequential (single-threaded). Appending `.parallelStream()` chunks the massive data set and processes it concurrently utilizing the CPU's default `ForkJoinPool`.

## Intermediate vs Terminal Operations in Streams
- **Intermediate Operations**: They return *another Stream*, allowing infinite chaining. Because of lazy evaluation, they do nothing until a terminal triggers the chain. 
  - *Examples*: `filter()`, `map()`, `sorted()`, `distinct()`.
- **Terminal Operations**: They return a non-stream physical result (like a `List`, `int`, or just `void`). Calling this immediately executes the entire chained pipeline.
  - *Examples*: `collect()`, `forEach()`, `reduce()`, `count()`.

## Immutability in Java
- **What it is**: An object is completely immutable if its physical state cannot mathematically be altered after it is constructed.
- **How to create securely**:
  1. Make the class `final` so it cannot be extended/overridden.
  2. Make all fields strictly `private final`.
  3. Provide absolutely zero `setter` methods.
  4. If the class holds a mutable object (like a `Date` or `List`), ALWAYS return a **deep clone** of it in your getters, never the original memory reference.
- **Benefits**: 100% thread-safe intrinsically, fantastic for caching, and hyper-safe for use as `HashMap` keys.

## Volatile keyword and thread visibility
- **The Problem**: Threads drastically boost performance by caching variables heavily into their local CPU Cache rather than slowly reading from RAM. This triggers a **Visibility Problem**: Thread A updates a variable, but Thread B never visually sees the change because Thread B is still reading from its own outdated CPU cache.
- **The Fix (`volatile`)**: Adding `volatile` to a variable forcefully commands every thread to read it DIRECTLY from Main Memory (RAM), aggressively bypassing the local CPU cache entirely.
- *Crucial Note:* `volatile` guarantees **visibility**, but it DOES NOT guarantee **atomicity** (you still absolutely need `AtomicInteger` to safely increment a number like `i++`).

## Transient keyword (Serialization)
- **What it is**: During Serialization (converting a Java object into a byte stream to send over a network socket or save to disk), any variable marked with `transient` is completely ignored and aggressively skipped.
- **Why use it?**: 
  - **Security/Privacy**: Passwords, SSNs, or highly secret API keys should never be hard-written to a local disk file.
  - **Logical Meaninglessness**: Variables that dynamically have no mathematical meaning if loaded later (like a current network socket connection or an open database session handle).

## Functional Interfaces and Lambda expressions
- **Functional Interface**: An explicitly simple interface that has precisely **one abstract method**. Usually marked with `@FunctionalInterface` (e.g., `Runnable`, `Callable`, `Comparator`).
- **Lambda Expressions**: Introduced heavily in Java 8, they are simply an ultra-short syntactic sugar shortcut precisely meant to provide an immediate implementation for that *single* abstract method without writing a gigantic boilerplate Anonymous Inner Class.
  - *Before*: `Runnable r = new Runnable() { public void run() { System.out.println("Hi"); } };`
  - *After Lambda*: `Runnable r = () -> System.out.println("Hi");`

## Shallow Copy vs Deep Copy
- **Shallow Copy**: Creates a brand new Object, but lazily copies over the exact same *memory references* inside of it. If Object A holds a pointer to a `List`, the clone Object B will point to that exact same shared `List`. If B modifies the `List`, A gets mutated too! (The default `clone()` method is Shallow).
- **Deep Copy**: Recursively duplicates *everything*. It creates a brand new Object A, and dynamically creates a brand new perfectly duplicated `List` to put inside it. Changes made natively to clone B have mathematically zero impact on Object A.

## Stack vs Heap Memory
- **Stack Memory**: Tiny, incredibly fast localized memory. It strictly securely stores primitive values (`int`, `boolean`) and exact memory *references* (pointers) to objects. Follows LIFO (Last-In-First-Out). Every individual thread physically gets its own isolated, completely private Stack.
- **Heap Memory**: Massive, universally shared global memory space utilized strictly for storing dynamically created Objects (`new Object()`, `new ArrayList()`). The Heap is inherently accessible by all threads (which is exactly why concurrency issues exist here) and is intelligently managed recursively by the **Garbage Collector**.

## Comparable vs Comparator
In Java, both interfaces are explicitly used for sorting collections (like an `ArrayList`), but they operate fundamentally differently:

### 1. Comparable (Natural Ordering)
- **What it is**: It defines the singular, "default" or "natural" way your object should be predictably sorted. 
- **Implementation**: It fundamentally modifies the **actual target class** natively. Your class physically implements `Comparable<T>` and explicitly overrides the `compareTo(T target)` method.
  - *Example*: `public class Employee implements Comparable<Employee> { ... }`
- **When to use**: When the object has an overwhelmingly obvious default sort parameter (like strictly sorting an Employee universally by their `employeeId`).
- **The Core Drawback**: You are hardcoding exactly **one** permanent sorting behavior into the class. If you use `Comparable`, you cannot magically sort Employees by descending Salary in one UI view and alphabetically by Name in another.

### 2. Comparator (Custom Ordering)
- **What it is**: A totally external sorting mechanism that defines a dynamic, **custom rule** for ordering.
- **Implementation**: It is written physically **outside** of your target class. It implements `Comparator<T>` and dictates the logic inside the `compare(T obj1, T obj2)` method.
  - *Modern Example*: `Comparator<Employee> bySalary = (e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary());`
- **When to use**: When you desperately need total dynamic flexibility. You can freely create 10 distinct `Comparator` objects (Sort by Age, Sort by Name, Sort by Start Date) without ever modifying the original `Employee` class source code.
- **Critical Logic Principle**: You **must** universally use a `Comparator` when you explicitly need to sort third-party classes generated from an external `.jar` file, precisely because you physically cannot edit their class files to add a `Comparable` interface!


## thenApply vs thenCombine vs thenCompose vs thenAccept
These are all chaining methods on **`CompletableFuture`** (Java 8+), used to build asynchronous pipelines. The key difference is *what they do with the result* of the previous stage:

### `thenApply()` — Transform the result
- **What it does**: Takes the result of the previous future, applies a function to it, and returns a **new** `CompletableFuture` with the transformed value. Think of it like `Stream.map()`.
- **When to use**: When you want to transform the value from Step A directly into a new value for Step B.
```java
CompletableFuture.supplyAsync(() -> "hello")
    .thenApply(s -> s.toUpperCase())   // transforms "hello" → "HELLO"
    .thenApply(s -> s + "!")           // transforms "HELLO" → "HELLO!"
    .thenAccept(System.out::println);  // prints "HELLO!"
```

### `thenCompose()` — Chain dependent futures (FlatMap)
- **What it does**: Used specifically when Step B is itself an **async function** that returns its own `CompletableFuture`. Without `thenCompose`, you'd end up with an ugly nested `CompletableFuture<CompletableFuture<User>>` that needs manual unwrapping. `thenCompose` automatically flattens it into `CompletableFuture<User>`.
- **When to use**: When each step **depends on** the result of the previous step AND is itself a non-blocking async call.

**Step 1 — Define the two async helper methods:**
```java
// Simulates a non-blocking DB call to fetch a user's ID
static CompletableFuture<Integer> fetchUserId(String username) {
    return CompletableFuture.supplyAsync(() -> {
        System.out.println("Fetching ID for: " + username);
        return 42; // pretend this came from DB
    });
}

// Simulates a non-blocking DB call that NEEDS the ID from the previous step
static CompletableFuture<String> fetchUserDetails(int userId) {
    return CompletableFuture.supplyAsync(() -> {
        System.out.println("Fetching details for userId: " + userId);
        return "User{id=42, name='Alice', role='Admin'}"; // pretend this came from DB
    });
}
```

**Step 2 — Chain them using `thenCompose`:**
```java
CompletableFuture<String> result = fetchUserId("alice")      // Step A: CompletableFuture<Integer>
    .thenCompose(id -> fetchUserDetails(id));                 // Step B: CompletableFuture<String> (flattened automatically)

result.thenAccept(System.out::println);
// Output:
// Fetching ID for: alice
// Fetching details for userId: 42
// User{id=42, name='Alice', role='Admin'}
```

**What would happen with `thenApply` instead?**
```java
// ❌ WRONG — produces CompletableFuture<CompletableFuture<String>> — a nested future you can't easily use
CompletableFuture<CompletableFuture<String>> nested = fetchUserId("alice")
    .thenApply(id -> fetchUserDetails(id));  // fetchUserDetails returns CF<String>, so thenApply wraps it again!
```

**What if you call `thenCompose` on a `CompletableFuture<Void>`?**
- `CompletableFuture<Void>` has no result value to pass forward (it's the return type of `thenAccept` and `runAsync`). 
-  If you chain `thenCompose` on it, your lambda receives `null` as the input. This is valid but only useful if your next step is a fire-and-forget action that doesn't need the previous result.
```java
CompletableFuture.runAsync(() -> System.out.println("Step A (no result)"))
    .thenCompose(voidResult -> CompletableFuture.runAsync(() -> System.out.println("Step B (voidResult is null)")));
```
> *In practice, chaining async work off a `Void` future should use `thenRun()` instead — it is cleaner as it explicitly signals "I expect no input from the previous step."*

**What if you call `thenCompose` on a `CompletableFuture<Integer>`?**
- This is the normal, intended usage. `thenCompose` receives the `Integer` result from the upstream future and passes it as the argument into your lambda. Your lambda **must** return a new `CompletableFuture<T>` (of any type), which is then flattened.
```java
CompletableFuture<Integer> step1 = CompletableFuture.supplyAsync(() -> 5);  // produces an Integer

CompletableFuture<String> step2 = step1.thenCompose(number -> {
    // 'number' here is the Integer 5, passed directly from step1
    return CompletableFuture.supplyAsync(() -> "The number doubled is: " + (number * 2));
});

step2.thenAccept(System.out::println);
// Output: The number doubled is: 10
```
- **Key point**: The `Integer` value flows naturally into the lambda as the typed argument. `thenCompose` unwraps the `CompletableFuture<Integer>`, hands you the raw `5`, and expects you to return a new `CompletableFuture` from your lambda in return. The final result is a clean, single `CompletableFuture<String>` — not a nested one.

> *In summary — `thenCompose` on a `CompletableFuture<T>` where `T` is a real value (like `Integer`, `String`, `User`) is the standard correct use case. The `Void` variant is the edge case where you're essentially signalling "I don't care about the previous result, just run this next."*

### `thenCombine()` — Merge two **independent** futures
- **What it does**: Runs two completely independent `CompletableFuture`s in parallel and combines their results once **both** complete.
- **When to use**: When Step A and Step B have no dependency on each other but you need both results together to continue.
```java
CompletableFuture<String> userFuture   = CompletableFuture.supplyAsync(() -> fetchUser());
CompletableFuture<String> priceFuture  = CompletableFuture.supplyAsync(() -> fetchPrice());

userFuture.thenCombine(priceFuture, (user, price) -> user + " owes: " + price)
          .thenAccept(System.out::println);
```

### `thenAccept()` — Consume the result (no return)
- **What it does**: Takes the result of the previous future and runs a side-effect action (like logging, writing to DB, printing). Returns `CompletableFuture<Void>` — it is a terminal operation in the chain.
- **When to use**: At the **end** of your pipeline when you want to do something with the final value but don't need to pass anything further.
```java
CompletableFuture.supplyAsync(() -> "Final Result")
    .thenAccept(result -> System.out.println("Got: " + result)); // terminal, returns void
```

### Quick Reference Summary

| Method | Input | Output | Use When |
|---|---|---|---|
| `thenApply` | `T` | `CompletableFuture<U>` | Transform a value (like map) |
| `thenCompose` | `T` | `CompletableFuture<U>` (flattened) | Chain dependent async calls (like flatMap) |
| `thenCombine` | `T` + `U` (parallel) | `CompletableFuture<V>` | Merge two independent futures |
| `thenAccept` | `T` | `CompletableFuture<Void>` | Terminal — consume result, no return |


## map vs flatMap

### `map()` — One-to-One transformation
- **What it does**: Applies a function to each element and wraps every single result into its own individual slot in the output Stream. The structure is **preserved** — one input element always produces exactly one output element.
- **Result**: `Stream<Stream<T>>` if your mapping function returns a collection (a nested stream!).

```java
List<String> words = List.of("Hello", "World");

// map → each word is split into a Stream<String[]>
words.stream()
     .map(word -> word.split(""))     // produces Stream<String[]> — a stream of ARRAYS
     .forEach(System.out::println);   // prints array references, not individual letters!
```

### `flatMap()` — One-to-Many, then Flatten
- **What it does**: Applies a function to each element (like `map`), but then **flattens** all inner Streams into a single, continuous output Stream. You go from `Stream<Stream<T>>` → `Stream<T>`.
- **Mental model**: "Map, then squash everything into one flat layer."

```java
List<String> words = List.of("Hello", "World");

// flatMap → each word is split AND the inner arrays are merged into one flat Stream
words.stream()
     .flatMap(word -> Arrays.stream(word.split(""))) // flattens into Stream<String>
     .forEach(System.out::print);                    // prints: HelloWorld
```

### What happens when you run `flatMap` on a 1D array (already flat)?
- `flatMap` strictly expects your mapping function to return a `Stream`. If you apply it on a flat/1D collection where each element maps to a **single-element stream**, it simply behaves identically to `map()` — no real flattening occurs because there is no nesting to collapse.

```java
List<Integer> numbers = List.of(1, 2, 3, 4);

// flatMap on a 1D list where mapper returns a single-element stream
numbers.stream()
       .flatMap(n -> Stream.of(n * 2))  // each element maps to Stream.of(one value)
       .forEach(System.out::print);     // prints: 2 4 6 8 — same as map(n -> n * 2)
```
> *The key takeaway: `flatMap` is only meaningfully different from `map` when each element produces a **multi-element stream** (like splitting a sentence into words, or a user into their list of orders). On a truly 1D flat structure with a single-value mapper, it is functionally equivalent to `map`.*

### Quick Comparison

| | `map` | `flatMap` |
|---|---|---|
| **Transformation** | 1 input → 1 output | 1 input → N outputs (flattened) |
| **Return type of mapper** | Any value `U` | `Stream<U>` |
| **Output Stream type** | `Stream<U>` | `Stream<U>` (flattened) |
| **Common use case** | Transforming values | Splitting/expanding nested structures |

## HashMap vs ConcurrentHashMap

### `HashMap`
- **Thread Safety**: Completely **not thread-safe**. If two threads simultaneously call `put()` on the same `HashMap`, they can corrupt its internal structure (infinite loops in Java 7 due to circular linked list during rehashing, or lost updates in Java 8+).
- **Null handling**: Allows **one `null` key** and **multiple `null` values**.
- **Performance**: Fastest in single-threaded contexts — zero locking overhead.
- **When to use**: Only in single-threaded code, or when you manage external synchronization yourself.

### `ConcurrentHashMap`
- **Thread Safety**: Fully thread-safe **without** locking the entire map. This is the key insight — it uses fine-grained locking so multiple threads can read and write simultaneously without blocking each other.
- **Null handling**: Does **NOT** allow `null` keys or `null` values. This is intentional — in a concurrent context, a `get()` returning `null` is ambiguous: does the key not exist, or was `null` explicitly stored? Disallowing it removes the ambiguity.
- **When to use**: Any shared, multi-threaded environment (caches, shared registries, etc.).

### How ConcurrentHashMap achieves thread safety internally

**Java 7 — Segment Locking:**
- The map was divided into fixed **Segments** (default: 16). Each segment was an independently locked mini-HashMap.
- Only the specific segment being written to was locked — other segments remained fully accessible.
- Max concurrency = number of segments (16 by default).

**Java 8+ — Node-level (CAS) Locking:**
- Segments were completely removed. Instead, locking happens at the individual **bucket/node level** using **Compare-And-Swap (CAS)** for lock-free reads and only locking a single bucket's head node for writes.
- This enables far greater concurrency — thousands of threads can write to different buckets simultaneously with no contention.

  **How CAS actually works (it's not a version — it's a value check):**
  - CAS is a single, atomic CPU-level instruction. It operates on three things: a **memory address**, the **expected current value**, and the **new value** you want to write.
  - The logic is: *"Only update this memory slot to the new value IF its current value still matches what I expect. If something else already changed it, do nothing and tell me it failed."*
  - In pseudocode:
    ```
    CAS(memoryAddress, expectedValue, newValue):
        if (*memoryAddress == expectedValue):
            *memoryAddress = newValue   // success — atomic swap
            return true
        else:
            return false                // someone already changed it, retry
    ```
  - In `ConcurrentHashMap`, when a thread wants to **insert into an empty bucket**, it reads the current head node (expected = `null`), and does a CAS: *"Set this bucket's head to my new node, only if it's still `null`."* If another thread already inserted something there, CAS returns `false` and the thread retries.
  - **Key difference from version-based locking (like Optimistic Locking in DB)**: Database Optimistic Locking attaches a version *number* to a row and increments it on each write. CAS directly compares the raw *value in memory* — no separate version counter needed. It's faster because it's a single atomic CPU instruction, not a DB round-trip.

### Quick Comparison

| Feature | `HashMap` | `ConcurrentHashMap` |
|---|---|---|
| **Thread-safe?** | ❌ No | ✅ Yes |
| **Locking** | None | Per-node (Java 8+) |
| **Null keys/values** | 1 null key, many null values | ❌ Neither allowed |
| **Performance (single-thread)** | Faster | Slight overhead |
| **Performance (multi-thread)** | Dangerous / broken | High concurrency |
| **Use case** | Single-threaded | Shared multi-threaded maps |
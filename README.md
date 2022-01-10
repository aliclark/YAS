# solo-pl
A simple close-to-the-metal object-oriented programming language

Currently in concept form, but if you're interested let me know and help build it :)

```
Hello(stdout, text)
{
   run() {
      stdout.write(text:"Hello, World!\n")
   }
}
```

# Aims

The language aesthetic could be described broadly by the phrases "economy of mechanism" and "low conceptual volume".

To this end, the language uses: composition, immutable variable definition, garbage collection, object-orientatated programming for encapsulating state, nil global state, structural typing with compile-time checks, plain data types with run-time checks, single-threaded execution and asynchronicity via promises, loop for iteration with iterable data type, outcome data type for handling error cases, pass by reference with the exception of a primitive integer type and primitive floating point type, accept parameters via method accessors on a supplied parameters object (no distinct concept of dictionary vs positional, or rest parameters vs. plain parameters).

Everything expressible in the language is expressed in terms of these concepts.

### To map closely to modern computer hardware:

* Reference counting could allow a minimum of memory to be allocated at any given moment,
  and also using a Last In First Out allocation to keep the addressed working set small.
* By frequently allocating and freeing objects in small lifetimes,
  the compiler could use peephole optimisation to use stack allocations instead.
* Aligning memory allocations to 16-byte boundaries, tagging the lowest 4-bits of a pointer/primitive
  could be used to indicate which of 16 possible data type options have occurred at runtime, to avoid extra boxing.
* Idiomatic use of simple data type matching may lend itself better to O(1) jump tables in conditionals,
  contrasted with if-else chains.

### Intended compilation targets at the moment are:

 * 64-bit ARM assembly (Graviton, M1, desktop and servers)
 * 64-bit x86 assembly (Intel, AMD, desktop and servers)
 * 32-bit C89 code (commodity embedded)
 * WASM (JavaScript and serverless environments)
 * TODO: GPU ? FPGA ?

```
| Conceptually, these should be thought as boring wrappers of their object
Optionally :: <result> none
Outcome :: <result> <error>

| queue, text, etc. are methods taking no parameters which are found on context instance.
Task(log, optionally, queue, text, integer, box) <context>
{
   | ditto but work and domain are on params
   prepare(work, domain) <params> {

      | boxes are handy for their mutability
      res = box.new()
      items = queue.new()

      | executes and matches work.assess()
      | the compiler determines whether work.assess() could return or have created a promise;
      | if it's possible then 'prepare' is also an async method which awaits on the work.access() call
      go work.assess() {
         result { res.set(result) }
         error { ok }
      }

      | executes and matches work.assess()
      | the compiler determines whether work.assess() only returns a promise;
      | if so then the matching occurs asynchronously (out of scope).
      | Otherwise it is a compile-time error.
      | Other promises could be created but the 'of' statement won't await on them.
      of work.assess() {
         | these blocks are somewhat like a closure
         | it is an error to use => return in these blocks;
         | that's too confusing because it is no longer in the prepare scope
         result {
            of work.triage(result) {
               result <deliverable> { work.plan(deliverable) }
               error { log.warn(error) }
            }
         }
         error { log.warn(error) }
      }

      | executes and matches work.assess()
      | the compiler determines whether work.assess() could return a promise;
      | if so then it is a compile-time error.
      | Other promises could be created but the 'if' statement won't await on them.
      if work.gather() {
         result { queue.add(result) }
         none { ok }
      }

      if integer.compare(items.length(), work.custom()) {
         less { => optionally.result(res) }
         greater equal { ok }
      }

      | always list the most common cases first; the compiler won't re-order them.
      | in the case of primitive like integer, "risks" and "priority" would refer to the same thing.
      | this is unlike user types where the former would be the constructed type, and the latter
      | would be the parameter for it
      if domain.challenges() <risks> {
         0x40..0xff 0x00..0x29 { => risks }
         0x30..0x39 <priority> { => integer.plus(priority, integer:1) }
      }

      | until does not await, so can't be used directly on promises
      | but you could iterate a list of promises and use an inner 'go' statement to await them.
      | in that case, until will await each loop in turn.
      until none work.gather() {
         result { queue.add(result) }
      }

      => text:"success"
   }
}
```

```
| user data type: defines one or more type constructors
| class: natively-defined class / user-defined class
| type constructor: a name for the constructor and whether a parameter is optionally accepted
| class constructor: an implicit method that accepts the context parameters and returns a new instance of the class
| instantiation of a type:
|   a constructed type whose constructor is runtime-discernable from the other constructors of the same type
|   using pattern matching.
|   if the constructor does not have a parameter,
|   then a default is used instead, which is an instance of a class which has no methods
| instance of a class
| thing: instance of a class / instantiation of a type / primitive
| user-defined class: 
|   defines parameters for the instance and its methods
| natively-defined class:
|   set of native functions to construct an instance and provide its methods.
|   also describes its interface to the compiler
| primitive: opaque primitive / instance primitive
| opaque primitive: integer (i60) / number (f60)
|   these may be used in pattern matching as ranges
| instance primitive:
|   any other parsed value, such as a text string or list
|   these may or may not be allowed for use in pattern matching
|   these may or may not have methods implemented
| pattern matching:
|   it is an error to pattern match on a return value which could (also) return something other
|   than an instantiation of type(s)
|   exactly all possible constructors must be matched against,
|   otherwise it is a compile time error.
|   it would be nice to allow a subset of matching,
|   but this adds too much syntactic and semantic complexity
|
| As an optimisation, instead of further boxing to construct a data type,
| using the top 4 bits of the pointer to indicate which of up to 4 boolean types,
| or 16 options of fewer types. Just shift right to get the pointer back.
| This assumes malloc alignment of 16 and minimum allocation is 2 longs.
|
| All variable assignments are immutable and local that specific block,
| and only visible from that point onwards within the same block (including sub-blocks)
| there is no shadowing allowed, including from pattern matching blocks

```

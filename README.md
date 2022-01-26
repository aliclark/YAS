# YAS

A simple close-to-the-metal object-oriented programming language

Currently in concept form, but if you're interested let me know and help build it :)

```
say Hello(stdout, text)
   def run()
      stdout.write(text:"Hello, World!\n")
   fed
yas
```

# Aims

The language aesthetic could be described broadly by the phrases "economy of mechanism" and "low conceptual volume".

To this end, the language uses: composition, immutable non-shadowable variables, garbage collection, object-orientatated programming for encapsulating state, nil global state, structural typing with compile-time checks, plain data types with run-time checks, single-threaded execution and asynchronicity via promises, loop for iteration with iterable data type, outcome data type for handling error cases, pass by reference with the exception of a primitive small integer type, pass parameters by position and/or by name, but no rest or dictionary parameters

Everything expressible in the language is expressed in terms of these concepts.

### To map closely to modern computer hardware:

* Small arena-based memory allocation and collection (a few MB is commensurate with current cache sizes)
* Classes are created at runtime to reduce the need for instance fields, reducing memory cache usage
* 32-bit references reduce memory cache usage
* Aligning memory allocations to 8-byte boundaries allows tagging the lowest 3-bits of a pointer/primitive
  could be used to indicate which of 8 possible data type options have occurred at runtime
* Idiomatic use of simple data type matching may lend itself better to O(1) jump tables in conditionals,
  compared to if-else chains.

### Intended compilation targets at the moment are:

 * 64-bit ARM assembly (Graviton, M1, desktop and servers)
 * 64-bit x86 assembly (Intel, AMD, desktop and servers)
 * Portable 32-bit/64-bit C89/C++ code (commodity embedded)
 * TypeScript (JavaScript and serverless environments)
 * GPU ? FPGA ?

It's a non-goal to create libraries for linking into other languages the only is to create application binaries only.

For simplicity the initial goal will be to first to write a compiler in TypeScript, which is very high level and should be quick to achieve.
Then port the compiler from TypeScript to Solo, using the TypeScript source code as a bootstrap.
Then using that compiler, target portable C code.
Then to start adding specialisations, eg. custom calling conventions in hand-coded assembly if the target is amd64 or aarch64.

### Example

```
| Conceptually, these are mutually-exclusive tags around an optional object parameter
Optionally :: <result> none
Outcome :: <result> <error>

say Task(log, optionally, queue, text, integer, box)

   def prepare(work, domain)

      | boxes are handy for their mutability
      res = box.new()
      items = queue.new()

      | executes and matches work.assess()
      | the compiler determines whether work.assess() could return or have created a promise;
      | if it's possible then 'prepare' is also an async method which awaits on the work.access() call
      | 'go' also creates a new memory arena which work.assess() and resulting code will allocate from
      | code in work.assess() can reference memory from prior arenas, but memory from prior arenas cannot
      | reference into a nested arena - instead memory must be copied or cloned backwards.
      go work.assess()
         result { res.set(result) }
         error { ok }
      og

      | executes and matches work.assess()
      | the compiler determines whether work.assess() only returns a promise;
      | if so then the matching occurs asynchronously (out of scope).
      | Otherwise it is a compile-time error.
      | Other promises could be created but the 'of' statement won't await on them.
      of work.assess()
         | these blocks are somewhat like a closure
         | it is an error to use => return in these blocks;
         | that's too confusing because it is no longer in the prepare scope
         result {
            of work.triage(result)
               result <deliverable> { work.plan(deliverable) }
               error { log.warn(error) }
            fo
         }
         error { log.warn(error) }
      fo

      | executes and matches work.assess()
      | the compiler determines whether work.assess() could return a promise;
      | if so then it is a compile-time error.
      | Other promises could be created but the 'if' statement won't await on them.
      if work.gather()
         result { queue.add(result) }
         none { ok }
      fi

      if integer.compare(items.length(), work.custom())
         less { => optionally.result(res) }
         greater equal { ok }
      fi

      | always list the most common cases first; the compiler won't re-order them.
      | in the case of primitive like integer, "risks" and "priority" would refer to the same thing.
      | this is unlike user types where the former would be the constructed type, and the latter
      | would be the parameter for it
      if domain.challenges() <risks>
         0x40..0xff 0x00..0x29 { => risks }
         0x30..0x39 <priority> { => integer.plus(priority, integer:1) }
      fi

      | until does not await, so can't be used directly on promises
      | but you could iterate a list of promises and use an inner 'go' statement to await them.
      | in that case, until will await each loop in turn.
      til none work.gather()
         result { queue.add(result) }
      lit

      => text:"success"
   fed

   def makeStuff(Foo, logger)
      | Foo is a "class template" (as is Task above)

      c1 = Foo.class()
      a = c1.new(log: logger, z: integer:1)
      b = c1.new(log: logger, z: integer:2)

      c2 = Foo.class(log: logger)
      x = c2.new(z: integer:1)
      y = c2.new(z: integer:2)

      | both of these sets of instances are functionally the same,
      | but the latter will have lower memory usage due to sharing logger in the context.
      | using this technique lowers the amount of data on the heap and by extension uses less cache memory
   fed
yas
```

### Terminology

```
| user data type: defines one or more type constructors
| class template: used to create a user-defined class by filling in zero or more context parameters
| class: natively-defined class / user-defined class
| type constructor: a name for the constructor, and designates whether a parameter is optionally accepted
| class constructor: an implicit method that accepts the instance parameters and returns a new instance of the class
| instantiation of a type:
|   a constructed type whose constructor is runtime-discernable from the other constructors of the same type
|   using pattern matching.
|   if the constructor does not have a parameter,
|   then a default is used instead, which is an instance of a class which has no methods
| instance of a class;
| thing: instance of a class / instantiation of a type / primitive
| user-defined class: 
|   defines parameters for the instance and its methods
| natively-defined class:
|   set of native functions to construct an instance and provide its methods.
|   also describes its interface to the compiler
| primitive: opaque primitive / instance primitive
| opaque primitive: integer (i31)
|   these may be used in pattern matching as ranges
| instance primitive:
|   any other parsed value, such as a text string or list
|   these may or may not be allowed for use in pattern matching
|   these may or may not have methods implemented
| pattern matching:
|   exactly all possible cases must be matched against,
|   otherwise it is a compile time error.
|   (it would be nice to allow only a subset of matching,
|   but this adds too much syntactic and semantic complexity)
```

# solo-pl
A simple close-to-the-metal object-oriented programming language

Currently in concept form, but if you are interested then please help me build it!

```
Hello(stdout, text)
{
   run() {
      stdout.write(text:"Hello, World!\n")
   }
}

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
| All variable assignments are local that specific block,
| and only visible from that point onwards within the same block (including sub-blocks)


| Conceptually, these should be thought as boring wrappers of their object
Optionally: <result> / none
Outcome: <result> / <error>

| queue, text, etc. are methods taking no parameters which are found on context instance.
Task(log, optionally, queue, text, value, box, cookie) <context>
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
      | the compiler determines whether work.assess() does return a promise;
      | if so then the matching occurs asynchronously (out of scope).
      | Otherwise it is a compile-time error.
      | Other promises could be created but the 'of' statement won't await on them.
      of work.assess() {
         | these blocks are somewhat like a closure
         | it is an error to use => return in these blocks; that's too confusing.
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

      if items.length().compareTo(work.custom()) {
         less { => optionally.result(res) }
         greater / equal { ok }
      }

      if domain.challenges() <risks> {
         0x00..0x29 / 0x40..0xff { => risks }
         0x30..0x39 <priority> { => priority.plus(integer:1) }
      }

      until none work.gather() {
         result { queue.add(result) }
      }

      => text:"success"
   }
}
```

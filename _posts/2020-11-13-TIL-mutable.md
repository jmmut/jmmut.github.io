---
tags: programming today-I-learned
excerpt_separator: <!--more-->
---

# mutable, const and caches

Have you ever done some kind of cache in C++? Like, computing a field lazily (so that it's not computed until it's needed), and keeping it to avoid computing it again.

And did you try to put that lazy field inside a class that is used as const in some place?

<!--more-->

You might see where I'm going. For interface purposes, if the operations you are doing are not modifying an object, you should be able to mark it as `const`. It shouldn't matter if that class internally computes something lazily, but you won't be able to call `const` methods if they write to *any* member field.

# Today I learned

... that you can mark a field as `mutable` and that will allow you to modify that field even if the enclosing class is marked as `const`.

> Mutable is used to specify that the member does not affect the externally visible state of the class (as often used for mutexes, memo caches, lazy evaluation, and access instrumentation).

```
class ThreadsafeCounter {
  mutable std::mutex m; // The "M&M rule": mutable and mutex go together
  int data = 0;
 public:
  int get() const {
    std::lock_guard<std::mutex> lk(m);
    return data;
  }
  void inc() {
    std::lock_guard<std::mutex> lk(m);
    ++data;
  }
};
```

# References

[https://en.cppreference.com/w/cpp/language/cv#mutable_specifier](https://en.cppreference.com/w/cpp/language/cv#mutable_specifier)

Polling pattern
===

When polling is inefficient
---
Basic multi-threading: 
walks thru a list of bits (each represents a resource, e.g. socket), and calls a handler/action when the bit is set.

The problem is the cost grows by O(N), where N is the total number of bits.

[http://www.wangafu.net/~nickm/libevent-book/01_intro.html](http://www.wangafu.net/~nickm/libevent-book/01_intro.html)
When comparing `select` vs `epoll`:
* `select` has to walk through the file descriptor (fd) list, thus O(N).
* For a single fd, `epoll` puts the fds it waits on into a list. When an event gets `epoll`-ed, the callback wakes up any waiters.

Thus, `epoll` scales better than O(N).

libevent provides unified API for different types of `epoll` (on different OSs)

node.js is implemented using libev, similar to libeven, which is great at async.
[https://www.geekgay.com/article-1430.html](https://www.geekgay.com/article-1430.html)

Is async threading always good?
---

* Completely async program can waste more memory space (and other resources due to competition).
* Too many callbacks can be complicated.

Coroutine
---
* non-preemptive, routines are suspended and resumed.
* e.g. python yeild, program proceeds from the last suspended point.
* gorouting: user-space scheduling, less cost than threads.



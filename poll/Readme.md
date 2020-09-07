Polling pattern
===

When polling is inefficient
---
Basic [multi-threading](http://www.wangafu.net/~nickm/libevent-book/01_intro.html):

walks thru a list of bits (each represents a resource, e.g. socket), and calls a handler/action when this bit is set.

The cost grows by O(N), where N is the total number of bits.

Comparing `select`/`poll` vs. `epoll`:
* `select`/`poll` has to walk through the file descriptor (fd) list, thus O(N) time.
* `epoll` prepares the set of fd that are "ready" for I/O ([man](https://man7.org/linux/man-pages/man7/epoll.7.html), [snippet](https://zhuanlan.zhihu.com/p/93609693)).
  * `epoll_create1`creates a new epoll instance.
  * `epoll_ctrl` adds items to the interest list of the epoll instance
  * If one resource in the interest list is ready, linux kernal calls a callback to move this resource into the ready list.
    * Notice that this step is done by kernel async.
  * `epoll_wait` returns the ready list if it's non-empty, otherwise blocks thread and sleeps.
 
Thus, `epoll` scales better, taking O(1) time.

[libuv](http://docs.libuv.org/en/v1.x/design.html) provides unified API for different types of `epoll` (on different OSs)
* How is event-loop implemented?
  * Circularly linked list libuv uses to store requests, handles and other dynamic stuff. [Example for understanding](https://gist.github.com/bodokaiser/5657156)


Is async threading always good?
---

* Completely async program can waste more memory space (and other resources due to competition).
* Too many callbacks can be complicated.

Coroutine
---
* non-preemptive, routines are suspended and resumed.
* e.g. python yeild, program proceeds from the last suspended point.
* gorouting: user-space scheduling, less cost than threads.

Further reading
---
[https://www.geekgay.com/article-1430.html](https://www.geekgay.com/article-1430.html)




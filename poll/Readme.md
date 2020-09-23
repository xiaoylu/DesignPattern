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
    * so CPU is not occupied, CPU can be used by other processes.
 
Thus, `epoll` scales better, taking O(1) time.

Event-loop
---
Naive web server:
* always creates a new thread whenever a new connection is built.
  * modern web servers need to deal with 10K clients concurrently (C10K problem).
  * max #threads is limited by hardward and software.

[libuv](http://docs.libuv.org/en/v1.x/design.html) provides unified API for different types of `epoll` (on different OSs)
* How is event-loop implemented?
  * Circularly linked list libuv uses to store requests, handles and other dynamic stuff. 
    * [Example for understanding](https://gist.github.com/bodokaiser/5657156)
    * [C Macro to locate any instance in memory](https://radek.io/2012/11/10/magical-container_of-macro/)
    * [Nice figure in this blog (scroll down)](https://blog.butonly.com/posts/node.js/libuv/1-libuv-overview/)
  * **Simplified** I/O poll step with `epoch`:
    * `uv__io_poll` iterates thru the `uv__io_t` struct chained on `loop->watcher_queue`
    * for each `uv__io_t`, add the `fd` it's been watching and pending events to the interest list via `epoll_ctl`
    * `epoll_wait` the events that's ready
    * for each events that's ready, call the `callback` associated with this event
    * In short, for each tick (superstep?)
      * process the current event loop
      * wait for the events that's ready
      * add new events to the loop to process in the next tick
    * A thread pool shared by different loops do the real time-consuming I/O work.
    
Coroutine
---

Is async threading always good?
* Completely async program can waste more memory space (and other resources due to competition).
* Too many callbacks can be complicated.

Coroutine:
* non-preemptive, routines are suspended and resumed.
* e.g. python yeild, program proceeds from the last suspended point.
* gorouting: user-space scheduling, less cost than threads.

Further reading
---
[https://www.geekgay.com/article-1430.html](https://www.geekgay.com/article-1430.html)




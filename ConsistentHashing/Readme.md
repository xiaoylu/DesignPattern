Consistent Hashing
===

Applications
---
* Distributed Hash Table (DHT) and Distributed Caching
  * maps the key to a location where the value is stored.

Idea
---
* Map cache servers to integers [0, 256] in a **ring**.
* Also map key to integers
  * the cache server with closest larger integer contains the key (hit the cache)

Load balancing
---
* when add cache server
  * new cache grabs the keys from the previous server
* when remove cache server
  * removed cache gives its keys to the next server
* virtual replicas
  * map one cache server to many integers


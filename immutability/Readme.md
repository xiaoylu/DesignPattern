Immutability
===

Immutability is good for a few reasons:

* When copying an object, the system simply returns a reference to the same object
* Only one copy of each distinct string value needs to be stored (called interning)
* Immutable objects are more thread-safe than mutable objects

[Static factory method](../factory/Readme.md) can be used to implement such interning

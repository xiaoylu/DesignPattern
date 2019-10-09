Immutability
===

Immutability is good for a few reasons:

* When copying an object, the system simply returns a reference to the same object
* Only one copy of each distinct string value needs to be stored (called interning)
* Immutable objects are more thread-safe than mutable objects

[Static factory method](../factory/Readme.md) can be used to implement such interning

Sometimes, if an object is mutable, you can create a wrapper (with private data member and no setter) to make it immutable.
For example, Java arrays are mutable, but `ByteString` which builds on top of the `byte[]` is immutable.

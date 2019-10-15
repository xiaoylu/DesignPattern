Views
===

In many cases, rather than copy the memory of an object,
we just need an alternative view of it (like another way to iterate through the collection).

For example, the `ImmutableSet` check if the `elements` actually is another `ImmutableSet`;
if yes, just create another **View** of the same object.

```java
// see https://guava.dev/releases/snapshot/api/docs/src-html/com/google/common/collect/ImmutableSet.html#line.218
public static <E> ImmutableSet<E> copyOf(Collection<? extends E> elements) {
  if (elements instanceof ImmutableSet && !(elements instanceof SortedSet)) {
    @SuppressWarnings("unchecked") // all supported methods are covariant
    ImmutableSet<E> set = (ImmutableSet<E>) elements;
    if (!set.isPartialView()) {
      return set;  // Another View!
    }
  } else if (elements instanceof EnumSet) {
    return copyOfEnumSet((EnumSet) elements);
  }
  Object[] array = elements.toArray();
  if (elements instanceof Set) {
    // assume probably no duplicates (though it might be using different equality semantics)
    return construct(array.length, array.length, array);
  } else {
    return constructUnknownDuplication(array.length, array);
  }
}
```

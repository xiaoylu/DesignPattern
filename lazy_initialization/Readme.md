Lazy Initialization
===

aka. delay the initialization of a field until its value is needed.

Lazy initialization holder
---
If this field is static, consider static lazy initialization
```
// Lazy initialization holder class idiom for static fields
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

// You do not need "synchronized" keyword to make 
// this function thread-safe!
private static FieldType getField() { return FieldHolder.field; }
```
The static field is not created until the fist time getField() gets called

because the static member is initialized the first time when it's used.

AND the subsequent access to the field does NOT involve any synchronization!

Double-check idiom
---

```
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field; // create local variable, read field only once
  if (result != null) // First check (no locking)
    return result;
  
  synchronized(this) {
      if (field == null) // Second check (with locking)
        field = computeFieldValue();
      return field;
    }
  }
```



Reference
---
"Effective Java" Item 83

Factory
===

Static Factory Method
---
Static Factory Method, unlike a public constructor,
* has customized names 
* create a new object **only if** necessary
* can return a subtype

For example, Java [Integer](http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/lang/Integer.java) has a cache for the integers in [-128,127].
When the argument int `i` is in this range, instead of creating a new `Integer`,
`Integer.valueOf(i)` return the cached `Integer(i)`.

```
// OpenJDK Integer.valueOf() implementation
// see http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/lang/Integer.java

public final class Integer extends Number implements Comparable<Integer> {
    
    ...
    // Static Factory Method
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    
    ...
    
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            high = ...;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
}

```

Python class method acts similary as the static method.

```python
from datetime import date

# random Person
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @staticmethod
    def fromFathersAge(name, fatherAge, fatherPersonAgeDiff):
        return Person(name, date.today().year - fatherAge + fatherPersonAgeDiff)

    @classmethod
    def fromBirthYear(cls, name, birthYear):
        return cls(name, date.today().year - birthYear)

    def display(self):
        print(self.name + "'s age is: " + str(self.age))

class Man(Person):
    sex = 'Male'

man = Man.fromBirthYear('John', 1985)
print(isinstance(man, Man)) # Return True

man1 = Man.fromFathersAge('John', 1965, 20)
print(isinstance(man1, Man)) # Return False
```

The difference between static method and class method is that:
* `cls` refers to the inheritted class (Man instead of Person)
* static method forces a hard-coded class (i.e. Person) at the time of creation

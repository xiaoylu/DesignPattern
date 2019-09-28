Factory
===

Static Factory Method
---
Static Factory Method, unlike a public constructor,
* has customized names 
* create a new object **only if** necessary
* can return a subtype

For example, Integer has a cache for the integers in [-128,127].
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


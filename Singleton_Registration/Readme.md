# Singleton And Registration Pattern

## Singleton
It helps to make sure a single copy of the instance exists.

```cc
// XSingleton is simply a wrapper of X which is optional.
struct XSingleton {
  X x;
  explict XSingleton() {
    x.init();
  }
};

// API to obtain the unique X.
X* getSingletonX() {
  static XSingleton singleton;
  return &singleton.x;
}

static X& first_call = (*getSingletonX());
```

## Registration Pattern
Use static initialization so that program can terminate early if unhealthy (rather than an error at later runtime)

In a header file, we define
```cc
#define REGISTER_MYCLASS(X)
  <put the singleton program above inside a macro>
```

And we can use this macro right after every defintion of X that should be a singleton.

## Static Global Initialization

DO NOT use global `static const std::string`!

A static member/local variable is initialized at *runtime* when the first time its definition was reached.

Such dynamic allocation is a bad practice because
- order undeterministic. another thread can access a static variable which has been destroyed.
- if this variable can not be trivally destroyed, then there could be memory leakage.

```
// BAD
namespace <global_namespace> {
static const char* a = "A";
static const std::string b = "B";
}

// GOOD
namespace <global_namespace> {
// Option 1.
static absl::string_view x = GetX();
absl::string_view GetX() {
  // local_x will never be destroyed.
  static std::string* local_x = new std::string("X");
  return *local_x;
}

// Option 2.
// compile time initialization; c++17 require only one copy of the inline global variable.
// string_view hold a `const char*`, which is not destroyed.
inline constexpr absl::string_vew y = "Y";
}
```

Note that primary types such as int, float does not apply here because they can be trivially destroyed.





Dependency Injection
===

Wikipedia:
> Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.

Dependency Injection is the new `new`.

Rather than looking up dependencies directly or from factories, the pattern recommends that dependencies are passed in. The process of setting dependencies into an object is called injection.

Dependency injection has two main benefits: it decouples classes and makes unit testing easier. 

The client `RealBillingService` does not construct its services `CreditCardProcessor` and `TransactionLog`.
Instead, the services are [passed to](https://github.com/google/guice/wiki/Motivation) the constructor of `RealBillingService`. 

```Java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
    // difficult to test code due to the compile-time dependency 
    // BAD: this.processor = new CreditCardProcessor();
    // BAD: this.transactionLog = new TransactionLog();
    
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

Guice is a dependency injection framework
---
* Declare @Inject constructors
```java
class FooService {
  private final MyDependency myDependency;

  @Inject
  FooService(MyDependency myDependency) {
    this.myDependency = myDependency;
  }
}
```

* Map types to their implementations.
```java

// option 1. bind to class
public class FooModule extends AbstractModule {
  @Override 
  protected void configure() {
     bind(Foo.class).to(SubclassFoo.class);
  }
}

// option 2. bind to instance (ThirdPartyFoo instance without @Injector)
public class FooModule extends AbstractModule {
  @Override
  public void configure() {
    bind(Foo.class).toInstance(ThirdPartyFoo.create());
  }
}

// option 3. bind via @Provides 
// Guice implicitly calls @Provides methods any time their 
// return type is requested with some @Inject constructor
// @Provides methods can have parameters that are injected by Guice

import com.google.inject.Provides;
...

public class PetModule extends AbstractModule {
  @Override
  public void configure() {
    bind(House.class).to(Mansion.class);
  }

  @Provides
  Pet providePet(House house) { return ThirdPartyCat.create(house); }
}

// option 3. notes
// @Provides methods will be called each time the dependency is requested, unless they are tagged with the 
// with @Singleton + @Provides, the Guice injector will only create a single instance and reuse it any time
```

* Runtime dependency instead of compile-time dependency

```java
public static void main(String[] args) {
  Injector injector = Guice.createInjector(new FooModule());
  FooService fooService = injector.getInstance(FooService.class);
  ...
}
```
When `getInstance(FooService.class)` gets called, Guice figures out the dependencies of `FooService` creates them and wires everything together.

* Lazy Creation via `Provider<>`
  * Declare `Provider<Pet> petProvider`
  * Lazily call `petProvider.get()` -- it finds out the `PetModule` maps `Pet` to `Cat`, so it returns a `Cat` object.
  ```java
  public class PetModule extends AbstractModule {
    @Override
    public void configure() {}
    
    @Provides
    Pet providePet() { return new Cat(); }
  }
  ```
  * You can control how many `Cat` are injected as `Pet` by calling `petProvider.get()` many times.

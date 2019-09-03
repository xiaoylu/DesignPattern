Design Pattern Notes
===

Dependency Injection
---

Wikipedia:
> Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.

Dependency Injection is the new `new`.

Rather than looking up dependencies directly or from factories, the pattern recommends that dependencies are passed in. The process of setting dependencies into an object is called injection.

The client `RealBillingService` does not construct its services `CreditCardProcessor` and `TransactionLog`.
Instead, the services are [passed to](https://github.com/google/guice/wiki/Motivation) the constructor of `RealBillingService`. 

```Java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
    // the direct, compile-time dependency is difficult to test
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

Guice
---
* Guice is a dependency injection framework
```java
class BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  BillingService(CreditCardProcessor processor, 
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    ...
  }
}
```

* map types to their implementations.
```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
     bind(TransactionLog.class).to(DatabaseTransactionLog.class);
     bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
  }
}
```

* when `getInstance(BillingService.class)` gets called, Guice figures out the dependencies of `BillingService`, i.e. `DatabaseTransactionLog` and `PaypalCreditCardProcessor`, creates them and wires everything together
```java
public static void main(String[] args) {
  Injector injector = Guice.createInjector(new BillingModule());
  BillingService billingService = injector.getInstance(BillingService.class);
  ...
}
```




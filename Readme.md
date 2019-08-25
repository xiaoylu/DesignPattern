Design Pattern Notes
===

Dependency Injection
---

Wikipedia:
> Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.

Dependency Injection is the new `new`.

The client `RealBillingService` does not construct its services `CreditCardProcessor` and `TransactionLog`.
Instead, the services are passed to the constructor of `RealBillingService`. 

```Java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
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



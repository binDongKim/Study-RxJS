# Core Concepts in RxJS - Part 2 - Subscription

## What is Subscription?

-  A Subscription is an object that represents a disposable/unsubscribable resource, usually the execution of an Observable.

- A Subscription has one important method, `unsubscribe`, that takes no argument and just disposes/unsubscribes the resource held by the subscription.

  ```javascript
  var observable = Rx.Observable.interval(1000);
  var subscription = observable.subscribe(x => console.log(x));
  // Later:
  subscription.unsubscribe();
  ```

- Subscriptions can also be put together, so that a call to an `unsubscribe()` of one Subscription may unsubscribe multiple Subscriptions. You can do this by *adding* one subscription into another:

  ```javascript 
  var observable1 = Rx.Observable.interval(400);
  var observable2 = Rx.Observable.interval(300);

  var subscription = observable1.subscribe(x => console.log('first: ' + x));
  var childSubscription = observable2.subscribe(x => console.log('second: ' + x));

  subscription.add(childSubscription);

  setTimeout(() => {
    // Unsubscribes BOTH subscription and childSubscription
    subscription.unsubscribe();
  }, 1000);
  ```

- Subscriptions also have a `remove(otherSubscription)` method, in order to undo the addition of a child Subscription.

## References

- [Subscription in RxJS](http://reactivex.io/rxjs/manual/overview.html#subscription)
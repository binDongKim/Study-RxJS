# Core Concepts in RxJS - Part 2 - Subject

## What is Subject?

-  An RxJS Subject is a special type of Observable that allows values to be multicasted to many Observers.

- While plain Observables are unicast (each subscribed Observer owns an independent execution of the Observable), Subjects are multicast.

- Subjects are like EventEmitters: They maintain a registry of many listeners.

- **Every Subject is an Observable:** Given a Subject, you can `subscribe` to it, providing an Observer, which will start receiving values normally. From the perspective of the Observer, it cannot tell whether the Observable execution is coming from a plain unicast Observable or a Subject.

  ```javascript
  var subject = new Rx.Subject();

  subject.subscribe({
    next: (v) => console.log('observerA: ' + v)
  });
  subject.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });

  subject.next(1);
  subject.next(2);
  ```

- Internally to the Subject, `subscribe` does not invoke a new execution that delivers values. It simply registers the given Observer in a list of Observers.

- **Every Subject is an Observer:** It is an object with the methods `next(v)`, `error(e)`, and `complete()`. To feed a new value to the Subject, just call `next(theValue)`, and it will be multicasted to the Observers registered to listen to the Subject. Since a Subject is an Observer, this also means you may provide a Subject as the argument to the `subscribe` of any Observable. With the approach below, we converted a unicast Observable execution to multicast through the Subject. This demonstrates how Subjects makes any Observable execution be shared to multiple Observers.

  ```javascript
  var subject = new Rx.Subject();

  subject.subscribe({
    next: (v) => console.log('observerA: ' + v)
  });
  subject.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });

  var observable = Rx.Observable.from([1, 2, 3]);

  observable.subscribe(subject); // You can subscribe providing a Subject
  ```

## Multicasted Observables

- A multicasted Observable uses a Subject under the hood to make multiple Observers see the same Observable execution.

- This is how the `multicast` operator works: Observers subscribe to an underlying Subject, and the Subject subscribes to the source Observable.

  ```javascript
  var source = Rx.Observable.from([1, 2, 3]);
  var subject = new Rx.Subject();
  var multicasted = source.multicast(subject);

  // These are, under the hood, `subject.subscribe({...})`:
  multicasted.subscribe({
    next: (v) => console.log('observerA: ' + v)
  });
  multicasted.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });

  // This is, under the hood, `source.subscribe(subject)`:
  multicasted.connect();
  ```

- `multicast` returns an Observable(ConnectableObservable Type) that looks like a normal Observable, but works like a Subject when it comes to subscribing.

- The `connect()` method is important to determine exactly when the shared Observable execution will start.

## Reference Counting

- Usually, we want to *automatically* connect when the first Observer arrives, and automatically cancel the shared execution when the last Observer unsubscribes.

- `refCount` method makes the multicasted Observable automatically start executing when the first subscriber arrives, and stop executing when the last subscriber leaves.

- When the number of subscribers of the multicasted Observable increases from `0` to `1`, it will call `connect()` for us, which starts the shared execution. Only when the number of subscribers decreases from `1` to `0` will it be fully unsubscribed, stopping further execution.

  ```javascript
  var source = Rx.Observable.interval(500);
  var subject = new Rx.Subject();
  var refCounted = source.multicast(subject).refCount();
  var subscription1, subscription2, subscriptionConnect;

  // This calls `connect()`, because
  // it is the first subscriber to `refCounted`
  console.log('observerA subscribed');
  subscription1 = refCounted.subscribe({
    next: (v) => console.log('observerA: ' + v)
  });

  setTimeout(() => {
    console.log('observerB subscribed');
    subscription2 = refCounted.subscribe({
      next: (v) => console.log('observerB: ' + v)
    });
  }, 600);

  setTimeout(() => {
    console.log('observerA unsubscribed');
    subscription1.unsubscribe();
  }, 1200);

  // This is when the shared Observable execution will stop, because
  // `refCounted` would have no more subscribers after this
  setTimeout(() => {
    console.log('observerB unsubscribed');
    subscription2.unsubscribe();
  }, 2000);
  ```

## Three Special Subjects

- **BehaviorSubject**

  - It stores the latest value emitted to its consumers.

  - Whenever a new Observer subscribes, it will immediately emit the *current value* to the Observer.

    ```javascript
    var subject = new Rx.BehaviorSubject(0); // 0 is the initial value

    subject.subscribe({
      next: (v) => console.log('observerA: ' + v)
    });

    subject.next(1);
    subject.next(2);

    subject.subscribe({
      next: (v) => console.log('observerB: ' + v)
    });

    subject.next(3);
    ```

    Output:

    ```javascript
    observerA: 0
    observerA: 1
    observerA: 2
    observerB: 2
    observerA: 3
    observerB: 3
    ```

    In the above example, the `BehaviorSubject` is initialized with the value `0` which the first Observer receives when it subscribes. The second Observer receives the value `2` even though it subscribed after the value `2` was sent.

- **ReplaySubject**

  - It's similar to `BehaviorSubject` in the way that it can send old values to new subscribers, but it can also *record* a part of the Observable execution.

  - It records multiple values from the Observable execution and replays them to new subscribers.

    ```Javascript
    // When creating a ReplaySubject, you can specify how many values to replay.
    var subject = new Rx.ReplaySubject(3);

    subject.subscribe({
      next: (v) => console.log('observerA: ' + v)
    });

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
      next: (v) => console.log('observerB: ' + v)
    });

    subject.next(5);
    ```

    Output:

    ```javascript
    observerA: 1
    observerA: 2
    observerA: 3
    observerA: 4
    observerB: 2
    observerB: 3
    observerB: 4
    observerA: 5
    observerB: 5
    ```

  - You can also specify a *window time* in milliseconds, besides of the buffer size, to determine how old the recorded values can be. 

    ```javascript
    var subject = new Rx.ReplaySubject(100, 500 /* windowTime */);

    subject.subscribe({
      next: (v) => console.log('observerA: ' + v)
    });

    var i = 1;
    setInterval(() => subject.next(i++), 200);

    setTimeout(() => {
      subject.subscribe({
        next: (v) => console.log('observerB: ' + v)
      });
    }, 1000);
    ```

    Output:

    ```javascript
    observerA: 1
    observerA: 2
    observerA: 3
    observerA: 4
    observerA: 5
    observerB: 3
    observerB: 4
    observerB: 5
    observerA: 6
    observerB: 6
    ...
    ```

    The second Observer gets events `3`, `4` and `5` that happened in the last `500` milliseconds prior to its subscription.

- **AsyncSubject**

  -  It only sents the *last* value of the Observable execution to its observers only when the execution *completes*.

    ```javascript
    var subject = new Rx.AsyncSubject();

    subject.subscribe({
      next: (v) => console.log('observerA: ' + v)
    });

    subject.next(1);
    subject.next(2);
    subject.next(3);
    subject.next(4);

    subject.subscribe({
      next: (v) => console.log('observerB: ' + v)
    });

    subject.next(5);
    subject.complete();
    ```

    Output:

    ```javascript
    observerA: 5
    observerB: 5
    ```

  - The AsyncSubject is similar to the `last` operator, in that it waits for the `complete` notification in order to deliver a single value.

## References

- [Subject in RxJS](http://reactivex.io/rxjs/manual/overview.html#subject)
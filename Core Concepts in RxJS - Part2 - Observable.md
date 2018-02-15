# Core Concepts in RxJS - Part2 - Observable

## What is Observable?

- Observables are lazy Push collections of multiple values.
- Observables are like functions with zero arguments, but generalize those to allow multiple values.
- Subscribing to an Observable is analogous to calling a Function.

## Core Observable concerns:

- **Creating Observables**

  - `Rx.Observable.create` operator is an alias for the `Observable` constructor, and it takes one argument: The _`subscribe` function_ (Note: here, the _`subscribe` function_ doesn't indicate Observable prototype `subscribe` function. It simply indicates the function which the `create` operator takes as an argument.)

    ```javascript
    var observable = Rx.Observable.create(function subscribe(observer) {
      var id = setInterval(() => {
        observer.next('hi')
      }, 1000);
    });
    ```

  - Observables can be created with `create`, but usually we use the so-called *creation operators*, like `of`, `from`, `interval`, etc.

- **Subscribing to Observables**

  - The _`subscribe` function_ calls are not shared among multiple Observers of the same Observable.
  - When calling `observable.subscribe` with an Observer, the  _`subscribe` function_ is run for that given Observer.

- **Executing the Observable**

  - The body of the _`subscribe` function_ represents an "Observable execution", a lazy computation that only happens for each Observer that subscribes.

  - There are three types of values an Observable Execution can deliver:

    - "Next" notification: sends a value such as a Number, a String, an Object, etc.
    - "Error" notification: sends a JavaScript Error or exception.
    - "Complete" notification: does not send a value.

  - In an Observable Execution, zero to infinite `Next` notifications may be delivered. If either an `Error` or `Complete` notification is delivered, then nothing else can be delivered afterwards.

    ```javascript
    var observable = Rx.Observable.create(function subscribe(observer) {
      try {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        observer.complete();
      } catch (err) {
        observer.error(err); // delivers an error if it caught one
      }
    });
    ```

- **Disposing/Unsubscribing Observables**

  - When `observable.subscribe` is called, it returns an object: the `Subscription`.

  - With `subscription.unsubscribe()`, you can cancel the ongoing execution:

    ```javascript
    var observable = Rx.Observable.from([10, 20, 30]);
    var subscription = observable.subscribe(x => console.log(x));
    // Later
    subscription.unsubscribe();
    ```

  - Each Observable must define how to dispose resources of that execution when we create the Observable using `create()`. You can do that by returning a custom `unsubscribe` function from within `function subscribe()`.

    ```javascript
    var observable = Rx.Observable.create(function subscribe(observer) {
      // Keep track of the interval resource
      var intervalID = setInterval(() => {
        observer.next('hi');
      }, 1000);

      // Provide a way of canceling and disposing the interval resource
      return function unsubscribe() {
        clearInterval(intervalID);
      };
    });
    ```

## References

- [Observable in RxJS](http://reactivex.io/rxjs/manual/overview.html#observable)
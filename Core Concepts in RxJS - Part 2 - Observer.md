# Core Concepts in RxJS - Part 2 - Observer

## What is Observable?

- An Observer is a consumer of values delivered by an Observable.

- Observers are simply a set of callbacks, one for each type of notification delivered by the Observable: `next`, `error`, and `complete`.

  ```javascript
  var observer = {
    next: x => console.log('Observer got a next value: ' + x),
    error: err => console.error('Observer got an error: ' + err),
    complete: () => console.log('Observer got a complete notification'),
  };
  ```

- To use the Observer, provide it to the `subscribe` of an Observable:

  ```javascript
  observable.subscribe(observer);
  ```

- Observers in RxJS may also be *partial*. If you don't provide one of the callbacks, the execution of the Observable will still happen normally, except some types of notifications will be ignored, because they don't have a corresponding callback in the Observer.

- When subscribing to an Observable, you may also just provide the callbacks as arguments, without being attached to an Observer object: (Note: If you provide the Observer with the form below, not an object format with key-value pair, the order of the callbacks matter, which means the callbacks would be considered as next - error - complete in order.)

  ```javascript
  observable.subscribe(
    x => console.log('Observer got a next value: ' + x),
    err => console.error('Observer got an error: ' + err),
    () => console.log('Observer got a complete notification')
  );
  ```

## References

- [Observer in RxJS](http://reactivex.io/rxjs/manual/overview.html#observer)
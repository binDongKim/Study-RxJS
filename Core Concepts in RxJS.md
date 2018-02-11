# Core Concepts in RxJS

## Three Features that make RxJS Powerful

- **Purity**: Producing values using pure functions.

  - Normal code

    ```javascript
    var count = 0;
    var button = document.querySelector('button');
    button.addEventListener('click', () => console.log(Clicked ${++count} times));
    ```


  - Using RxJS

    ```javascript
    var button = document.querySelector('button');
    Rx.Observable.fromEvent(button, 'click')
      .scan(count => count + 1, 0)
      .subscribe(count => console.log(`Clicked ${count} times`));
    ```

- **Flow**: The whole range of operators that helps you control how the events flow.

  - Normal code

    ```javascript
    var count = 0;
    var rate = 1000;
    var lastClick = Date.now() - rate;
    var button = document.querySelector('button');
    button.addEventListener('click', () => {
      if (Date.now() - lastClick >= rate) {
        console.log(`Clicked ${++count} times`);
        lastClick = Date.now();
      }
    });
    ```

  - Using RxJS

    ```javascript
    var button = document.querySelector('button');
    Rx.Observable.fromEvent(button, 'click')
      .throttleTime(1000)
      .scan(count => count + 1, 0)
      .subscribe(count => console.log(`Clicked ${count} times`));
    ```

- **Values**: Transform the values passed through observables.

  - Normal code

    ```javascript
    var count = 0;
    var rate = 1000;
    var lastClick = Date.now() - rate;
    var button = document.querySelector('button');
    button.addEventListener('click', (event) => {
      if (Date.now() - lastClick >= rate) {
        count += event.clientX;
        console.log(count)
        lastClick = Date.now();
      }
    });
    ```

  - Using RxJS

    ```javascript
    var button = document.querySelector('button');
    Rx.Observable.fromEvent(button, 'click')
      .throttleTime(1000)
      .map(event => event.clientX)
      .scan((count, clientX) => count + clientX, 0)
      .subscribe(count => console.log(count));
    ```



## The Essential Members in RxJS World 

- Observable
- Observer
- Subscription
- Operators
- Subject
- Schedulers



## References

- [RxJS Documentation v4](https://github.com/Reactive-Extensions/RxJS/tree/master/doc)
- [RxJS Overview](http://reactivex.io/rxjs/manual/overview.html)
# Core Concepts in RxJS - Part 2 - Operators

## What are Operators?

- Operators are the essential pieces that allow complex asynchronous code to be easily composed in a declarative manner.

- Operators are **methods** on the Observable type, such as `.map(...)`, `.filter(...)`, `.merge(...)`.

- When called, they do not *change* the existing Observable instance. Instead, they return a *new* Observable.

- An Operator is a function which creates a new Observable based on the current Observable. This is a pure operation: the previous Observable stays unmodified.

- An Operator is essentially a pure function which takes one Observable as input and generates another Observable as output. *Subscribing to the output Observable will also subscribe to the input Observable.*

  ```javascript
  function multiplyByTen(input) {
    var output = Rx.Observable.create(function subscribe(observer) {
      input.subscribe({
        next: (v) => observer.next(10 * v),
        error: (err) => observer.error(err),
        complete: () => observer.complete()
      });
    });
    return output;
  }

  var input = Rx.Observable.from([1, 2, 3, 4]);
  var output = multiplyByTen(input);
  output.subscribe(x => console.log(x));
  ```

  Output:

  ```javascript
  10
  20
  30
  40
  ```

  *Notice that a subscribe to `output` will cause `input` Observable to be subscribed. We call this an "operator subscription chain".*

## Instance Operators vs Static Operators

- **Instance Operators**

  - Instance Operators are methods on Observable instances, which means they are implemented on the prototype object of Observable class.

    ```javascript
    Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
      var input = this;
      return Rx.Observable.create(function subscribe(observer) {
        input.subscribe({
          next: (v) => observer.next(10 * v),
          error: (err) => observer.error(err),
          complete: () => observer.complete()
        });
      });
    }
    ```

    *Notice how the `input` Observable is not a function argument anymore, it is assumed to be the `this` object.* And Instance Operators would be used like below:

    ```javascript
    var observable = Rx.Observable.from([1, 2, 3, 4]).multiplyByTen();

    observable.subscribe(x => console.log(x));
    ```

- **Static Operators**

  - Static operators are pure functions attached to the Observable class directly, and usually are used to create Observables from *scratch*.

  - The most common type of static operators are the so-called *Creation Operators*. Instead of transforming an input Observable to an output Observable, they simply take a *non-Observable* argument and create a new Observable.

    ```javascript
    var observable = Rx.Observable.fromEvent(button, 'click');
    ```

## Which Operator do I use?

[Choose an operator](http://reactivex.io/rxjs/manual/overview.html#choose-an-operator) section in RxJS docs covers which operator you are likely to use in a various of cases. Get some help if you are struggle with specifying the case you are facing.

## Categories of Operators

- [Creation Operators](http://reactivex.io/rxjs/manual/overview.html#creation-operators)
- [Transformation Operators](http://reactivex.io/rxjs/manual/overview.html#transformation-operators)
- [Filtering Operators](http://reactivex.io/rxjs/manual/overview.html#filtering-operators)
- [Combination Operators](http://reactivex.io/rxjs/manual/overview.html#combination-operators)
- [Multicasting Operators](http://reactivex.io/rxjs/manual/overview.html#multicasting-operators)
- [Error Handling Operators](http://reactivex.io/rxjs/manual/overview.html#error-handling-operators)
- [Utility Operators](http://reactivex.io/rxjs/manual/overview.html#utility-operators)
- [Conditional and Boolean Operators](http://reactivex.io/rxjs/manual/overview.html#conditional-and-boolean-operators)
- [Mathematical and Aggregate Operators](http://reactivex.io/rxjs/manual/overview.html#mathematical-and-aggregate-operators)

## Pipeable Operators(Since v5.5)

### What is Pipeable Operator?

- It's basically any function that returns a function with the signature:

  `<T, R>(source: Observable<T>) => Observable<R>`

- There's a `pipe` method built into Observable class which can be used to compose the *Pipeable Operators*. 

- There's another `pipe` utility function at `rxjs/util/pipe` that can be used to build reusable *Pipeable Operators* from other *Pipeable Operators*. 

### How to use?

You just import any operators you need from `rxjs/operators` and *pipe* them by using `pipe` method.

```javascript
import { range } from 'rxjs/observable/range';
import { map, filter, scan } from 'rxjs/operators';

const source$ = range(0, 10);

source$.pipe(
  filter(x => x % 2 === 0),
  map(x => x + x),
  scan((acc, x) => acc + x, 0)
)
.subscribe(x => console.log(x))
```

> Note: The names of the pipeable version of some operators have changed, which are:
>
> - do -> tap
> - catch -> catchError
> - switch -> switchAll
> - finally -> finalize

### Why Pipeable Operators over Patched Operators?

- They are *tree-shakeable* by tools like webpack because they are just functions pulled in from modules directly, whereas importing Patched Operators would simply patch the global Observable so the tools wouldn't be able to detect/tell whether or not the operators are used which might lead to an unnecessary, unused operators being part of the output bundle.
- Pipeable Operators make *Functional composition* possible. Build your own custom operators and just pipe them with `pipe` operator.

### Known Issues

If you have no control over build process, like using Angular, or unable to upgrade to Webpack 3+, importing from `rxjs/operators` would likely make your application bundle larger. So instead, you would have to use deep imports:

```javascript
import { map } from 'rxjs/operators/map';
import { filter } from 'rxjs/operators/filter';
import { reduce } from 'rxjs/operators/reduce';
```

You can find more info about why this issue occurs and the other known issues in the reference below.

## References

- [Operators in RxJS](http://reactivex.io/rxjs/manual/overview.html#operators)
- [Build Your Own Operators Easily](https://github.com/ReactiveX/RxJS/blob/master/doc/pipeable-operators.md#build-your-own-operators-easily)
- [Piping all Operators in RxJS](https://blog.hackages.io/rxjs-5-5-piping-all-the-things-9d469d1b3f44)
- [Understanding Pipeable Operators](https://blog.angularindepth.com/rxjs-understanding-lettable-operators-fe74dda186d3)
- [Known issues in using Pipeable Operators](https://github.com/ReactiveX/RxJS/blob/master/doc/pipeable-operators.md#known-issues)
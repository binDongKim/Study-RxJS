# Contents

## Combination

- [combineLatest](#combinelatest)
- [withLatestFrom](#withlatestfrom)
- [zip](#zip)

## Conditional

## Creation

## Error Handling

## Filtering

- [skip](#skip)

## Multicasting

## Transformation

- [pluck](#pluck)
- [mergeMap](#mergemap)

## Utility



## combineLatest

<img src="http://reactivex.io/rxjs/img/combineLatest.png" alt="combineLatest" style="width: 550px; height: 300px">

**Signature**: `combineLatest(observables: …Observable, project: function): Observable`

`combineLatest` combines the most recent values from all the Observables passed as arguments. Whenever any Observable emits, it collects the most recent values from each Observable and emits those values as a form of *array* **unless** you transform it with the *project* function. If any input Observable errors, `combineLatest` will error immediately as well, and all other Observables will be unsubscribed. 

Note: `combineLatest` will not emit an initial value until each observable emits at least one value. 

If you are working with observables that only emit one value, or you only require the last value of each before completion, `forkJoin` is likely a better option.

- Example 1.

```javascript
const firstTimer$ = timer(0, 1000); // emit 0, 1... after every second, starting from now
const secondTimer$ = timer(500, 1000); // emit 0, 1... after every second, starting 0.5s from now
const combinedTimers = combineLatest(firstTimer$, secondTimer$);

combinedTimers.subscribe(value => console.log(value));
// Logs
// [0, 0] after 0.5s
// [1, 0] after 1s
// [1, 1] after 1.5s
// [2, 1] after 2s
```

- Example 2.

```javascript
const weight$ = of(70, 72, 76, 79, 75);
const height$ = of(1.76, 1.77, 1.78);
const bmi = combineLatest(weight$, height$, (w, h) => w / (h * h));

bmi.subscribe(x => console.log('BMI is ' + x));
// Logs
// BMI is 24.212293388429753
// BMI is 23.93948099205209
// BMI is 23.671253629592222
```



## pluck

<img src="http://reactivex.io/rxjs/img/pluck.png" alt="combineLatest" style="width: 600px; height: 300px">

**Signature**: `pluck(properties: ...string): Observable`

`pluck` *maps* each source value to the specified (nested) property. The given string values describe the path to the object(the source value) and `pluck` retrieves the value of the specified (nested) property from all values in the source Observable. If a property can't be resolved, it will return `undefined` for that value.

What makes `map` operator different from `pluck` operator is that you can perform an operation with it like,

```javascript
.map(user => user.age > 18 ? 'major': 'minor')
```

Whereas you just take a value with `pluck`. It simply picks one of the nested properties of each emitted value. 

One convenient feature `pluck` offers is that you can traverse the object tree without having to be worried for falling into referencing `null` property. 

- Example 1.

```javascript
const source$ = from([
  { name: 'Joe', age: 30, job: { title: 'Developer', language: 'JavaScript' } },
  { name: 'Sarah', age: 35 }
]);
const example = source.pipe(pluck('job', 'title'));

exmaple.subscribe((val) => console.log(val));
// Logs
// Developer
// undefined
```

- Example 2.

```javascript
const clicks = Rx.Observable.fromEvent(document, 'click');
const tagNames = clicks.pluck('target', 'tagName');

tagNames.subscribe(x => console.log(x));
```



## mergeMap

<img src="http://reactivex.io/rxjs/img/mergeMap.png" style="width: 600px; height: 300px">

**Signature**: `mergeMap(project: function: Observable, resultSelector: function: any, concurrent: number): Observable`

The difference between `switchMap` and `mergeMap ` : 

- When using `switchMap`, each inner subscription is completed when the source emits, allowing only one active inner subscription.
- In contrast, `mergeMap` allows for multiple inner subscriptions to be active at the same time. 

Note: If order must be maintained, `concatMap` could be the right choice.

You can limit the number of active inner subscriptions at a time with the `concurrent` parameter.

- Example 1.

```javascript
const letters = of('a', 'b', 'c');
const result = letters.pipe(
    mergeMap(x => interval(1000).map(i => x+i))
);

result.subscribe(x => console.log(x));
// Logs
// a0
// b0
// c0
// a1
// b1
// c1
// continues to list a,b,c with respective ascending integers
```



## skip

<img src="http://reactivex.io/rxjs/img/skip.png" style="width: 600px; height: 300px">

**Signature**: `skip(count: Number): Observable`

`skip` *skips/ignores* the first provided number of emissions from the source. It's useful when you have an observable that emits certain values on subscription that you wish to ignore. The likely scenario is that you are subscribing to a `Replay` or `BehaviorSubject` and do not need the initial values they emit. You could mimic `skip` by using `filter` with indexes: `filter((val, index) => index > 1)`

- Example 1.

```javascript
import { interval } from 'rxjs/observable/interval';
import { skip } 'rxjs/operators';

//emit every 1s
const source = interval(1000);
//skip the first 5 emitted values
const example = source.pipe(skip(5));
//output: 5...6...7...8........
const subscribe = example.subscribe(val => console.log(val));
```



## withLatestFrom

<img src="http://reactivex.io/rxjs/img/withLatestFrom.png" style="width: 600px; height: 300px">

**Signature**: `withLatestFrom(other: Observable, project: function): Observable`

`withLatestFrom` combines each value from the source Observable with the latest values from the other(input) Observables only when the source Observable emits a value. All the input Observables must emit at least one value for the output Observable to emit a value.

The difference between `combineLatest` and `withLatestFrom`:

- The output Observable from `withLatestFrom` only emits when the **source** Observable emits a value.
- In contrast, the one from `combineLatest` emits values any of input Observables emits a value.

`project` function can be passed to combine values. It receives the values in order of the passed Observable, where the first parameter is the value from the source Observable. (e.g. `a$.withLatestFrom(b$, c$, (a1, b1, c1) => a1 + b1 + c1)`). If `project` function is not passed, arrays will be emitted on the ouput Observable.

- Example 1.

```javascript
import { withLatestFrom, map } from 'rxjs/operators';
import { interval } from 'rxjs/observable/interval';

//emit every 5s
const source = interval(5000);
//emit every 1s
const secondSource = interval(1000);
const example = source.pipe(
  withLatestFrom(secondSource),
  map(([first, second]) => {
    return `First Source (5s): ${first} Second Source (1s): ${second}`;
  })
);
/*
  "First Source (5s): 0 Second Source (1s): 4"
  "First Source (5s): 1 Second Source (1s): 9"
  "First Source (5s): 2 Second Source (1s): 14"
  ...
*/
const subscribe = example.subscribe(val => console.log(val));
```



## zip

<img src="http://i1.wp.com/adamborek.com/wp-content/uploads/2017/05/zip_diagram.png?w=1080" style="width: 600px; height: 300px">

**Signature**: `zip(observables: *, project: function): Observable`

`zip` emits values as an array after all the input observables emit. The difference between `zip` and other combine operators such as `combineLatest`, `withLatestFrom` is that `zip` waits until there is a new value from each stream as you can see from the above diagram. In other words, `zip` always creates pairs from events with the same indexes (ex. 1A, 2B, 3C, 4D). This will continue until at least one inner observable completes.

- Example 1.

```javascript
import { delay } from 'rxjs/operators';
import { of } from 'rxjs/observable/of';
import { zip } from 'rxjs/observable/zip';

const sourceOne = of('Hello');
const sourceTwo = of('World!');
const sourceThree = of('Goodbye');
const sourceFour = of('World!');
//wait until all observables have emitted a value then emit all as an array
const example = zip(
  sourceOne,
  sourceTwo.pipe(delay(1000)),
  sourceThree.pipe(delay(2000)),
  sourceFour.pipe(delay(3000))
);
//output: ["Hello", "World!", "Goodbye", "World!"]
const subscribe = example.subscribe(val => console.log(val));
```

- Example 2.

```javascript
import { take } from 'rxjs/operators';
import { interval } from 'rxjs/observable/interval';
import { zip } from 'rxjs/observable/zip';

//emit every 1s
const source = interval(1000);
//when one observable completes no more values will be emitted
const example = zip(source, source.pipe(take(2)));
//output: [0,0]...[1,1]
const subscribe = example.subscribe(val => console.log(val));
```


# Contents

## Combination

- [combineLatest](#combinelatest)

## Conditional

## Creation

## Error Handling

## Filtering

## Multicasting

## Transformation

- [pluck](#pluck)

## Utility

---

### combineLatest

<img src="http://reactivex.io/rxjs/img/combineLatest.png" alt="combineLatest" style="width: 550px; height: 300px">

**Signature**: `combineLatest(observables: …Observable, project: function): Observable`

`combineLatest` combines the most recent values from all the Observables passed as arguments. Whenever any Observable emits, it collects the most recent values from each Observable and emits those values as a form of *array* **unless** you transform it with the *project* function. If any input Observable errors, `combineLatest` will error immediately as well, and all other Observables will be unsubscribed. Note: `combineLatest` will not emit an initial value until each observable emits at least one value. If you are working with observables that only emit one value, or you only require the last value of each before completion, `forkJoin` is likely a better option.

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



### pluck

<img src="http://reactivex.io/rxjs/img/pluck.png" alt="combineLatest" style="width: 600px; height: 300px">

**Signature**: `pluck(properties: ...string): Observable`

`pluck` *maps* each source value to the specified (nested) property. The given string values describe the path to the object(the source value) and `pluck` retrieves the value of the specified (nested) property from all values in the source Observable. If a property can't be resolved, it will return `undefined` for that value.

The difference from `map` operator is that you can perform an operation with `map` like,

```javascript
.map(user => user.age > 18 ? 'major': 'minor')
```

which you can't do with `pluck`. It simply picks one of the nested properties of each emitted value. A convenient feature `pluck` offers is that you can traverse the object tree without having to be worried for falling into referencing `null` property. 

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


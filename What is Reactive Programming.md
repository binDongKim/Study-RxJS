# What is Reactive Programming?

## Core Concepts

- Reactive programming is programming with asynchronous data streams.
- Anything can be considered as a stream.
- A stream can be used as an input to another one. 
- Operators/functions return a **new stream** based on the original stream. It does not modify the original click stream in any way. This is a property called **immutability**.
- You can *merge* two streams. You can *filter* a stream to get another one that has only those events you are interested in. You can *map* data values from one stream to another new one.



## Pull vs Push

> *Pull* and *Push* are two different protocols that describe how a data *Producer* can communicate with a data *Consumer*.

- Pull / Synchronous / Interactive
  - The Consumer determines when it receives data from the data Producer. The Producer itself is unaware of when the data will be delivered to the Consumer.
  - Every JavaScript Function is a Pull system. The function is a Producer of data, and the code that calls the function is consuming it by "pulling" out *single/multiple* return value/values from its call.
  - The application actively polls a data source for more information by retrieving data from a sequence that represents the source.
  - The application is active in the data retrieval process: it decides about the pace of the retrieval by calling `next` at its own convenience.
- Push / Asynchronous / Reactive
  - The Producer determines when to send data to the Consumer. The Consumer is unaware of when it will receive that data.
  - RxJS introduces Observables, a new Push system for JavaScript. An Observable is a Producer of multiple values, "pushing" them to Observers (Consumers).
  - The application gets information(data) by subscribing to a data stream (called observable sequence in RxJS), and any update is handed to it from the source.
  - The application is passive in the data retrieval process: apart from subscribing to the observable source, it does not actively poll the source, but merely react to the data being pushed to it.



## Observable in Reactive Programming

- Observable can emit three different things: a value (of some type), an error, or a "completed" signal.

- Consider that the "completed" takes place, for instance, when the current window or view containing that button is closed.

- We capture these emitted events only **asynchronously**, by defining a function that will execute when a value is emitted, another function when an error is emitted, and another function when 'completed' is emitted.

- The "listening" to the stream is called **subscribing**. The functions we are defining are **observers**. The stream is the **subject** (or **observable**) being observed.

  ​

## Features of Observable that differs from Promise

- Cancellable
- Lazy: Only called when we subscribe it
- Keeps emitting values over time
- Offers a various of operators



## References

- [Introduction to Reactive Programming](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [RxJS Overview](http://reactivex.io/rxjs/manual/overview.html)
- [Promise vs Observable](https://chrisnoring.gitbooks.io/rxjs-5-ultimate/content/observable.html)
# TS-Result

This tiny library provides a set of utility types and functions for handling results of computations that may succeed or fail. It is designed to make error handling more explicit and safer by wrapping results in a `Result` type similar to [Rust](https://doc.rust-lang.org/std/result/).

## Types

- `ErrResult<E>`: Represents the result of an asynchronous function that failed.
- `OkResult<D>`: Represents the result of an asynchronous function that succeeded.
- `Result<D, E>`: Represents the result of an asynchronous function that may succeed or fail.
- `PromiseResult<D, E> = Promise<Result<D, E>>`

## Functions

- `ok(val)`: Creates an ok result.
- `err(message)`: Creates an err result with the given error message.
- `err(error)`: Creates an err result with the given error.
- `err(ErrorClass, ...args)`: Creates an err result with the given `Error` subclass and arguments.
- `tryCatch(expression, errorTransformer)`: Wraps a synchronous or asynchronous computation that may throw an error to return a result object. Expression should be an inline callback function to ensure the this context is correct, thus the callback is not designed to take arguments.

## Usage

```typescript
import { ok, err, tryCatch } from './index';

// Creating an ok result
const success = ok('test');
console.log(success); // { ok: true, val: 'test', err: undefined }

// Creating an err result with an error
const failure = err(new Error('test'));
console.log(failure); // { ok: false, val: undefined, err: Error: test }

// Creating an err result with an error class
class TestError extends Error {
  name = 'TestError' as const;
  constructor(
    message: string,
    public readonly cause: Error,
  ) {
    super(message);
  }
}
const failureWithCustomError = err(TestError, 'test', new Error('cause'));
console.log(failureWithCustomError); // { ok: false, val: undefined, err: TestError: test }

// Creating an err result with a string
const failureWithString = err('test');
console.log(failureWithString); // { ok: false, val: undefined, err: Error: test }

// Using tryCatch with a synchronous function
const resultSync = tryCatch(() => 'test');
console.log(resultSync); // { ok: true, val: 'test', err: undefined }

// Using tryCatch with an asynchronous function
const resultAsync = await tryCatch(async () => Promise.resolve('test'));
console.log(resultAsync); // { ok: true, val: 'test', err: undefined }

// Using tryCatch with a synchronous function that throws
const resultSyncThrow = tryCatch(() => {
  throw new Error('test');
});
console.log(resultSyncThrow); // { ok: false, val: undefined, err: Error: test }

// Using tryCatch with an asynchronous function that rejects
const resultAsyncReject = await tryCatch(async () => Promise.reject(new Error('test')));
console.log(resultAsyncReject); // { ok: false, val: undefined, err: Error: test }

// Using tryCatch with a function that throws a non-error
const resultNonError = await tryCatch(async () => Promise.reject('test'));
console.log(resultNonError); // { ok: false, val: undefined, err: Error: Unexpected throw value 'test' of type string }

// Using tryCatch with a custom error transformer
class AnticipatedError extends Error {
  name = 'AnticipatedError' as const;
}
const errorTransformer = (e: Error) => {
  if (e.message === 'expected error') return new AnticipatedError();
  throw e;
};
const resultCustomError = await tryCatch(() => Promise.reject(new Error('expected error')), errorTransformer);
console.log(resultCustomError); // { ok: false, val: undefined, err: AnticipatedError }
```

## Installation

To install this library, run:

```bash
yarn add ts-result
```
or 

```bash
npm install --save ts-result
```


## License

This library is released under the MIT license.
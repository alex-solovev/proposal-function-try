# Function.prototype.try for JavaScript

## Rationale
In JavaScript, error handling typically relies on try-catch blocks, which can lead to more verbose code, especially when dealing with asynchronous operations. By introducing `Function.prototype.try`, developers can manage errors in a structured and consistent format, similar to the approach seen in Go. This method allows errors to be treated as regular return values, which encourages more robust error handling practices and prevents errors from being overlooked. The introduction of Function.prototype.try would streamline error management, making the code cleaner and more maintainable.

## Description
The proposed `Function.prototype.try` method encapsulates function execution within an implicit try-catch block, returning a `Result` object that contains the fields `value`, `error`, and an `isError` boolean flag. For asynchronous operations, the method returns a `Promise` that resolves to a `Result` object, ensuring that error handling remains consistent across both synchronous and asynchronous contexts. This feature is particularly beneficial in scenarios involving multiple asynchronous operations, as it allows each operation's errors to be handled distinctly and directly.

```js
const result = JSON.parse.try("}"); // no destructuring
const { error, value, isError } = JSON.parse.try("}"); // object destructuring
const [err, json, isErr] = await fetch.try("http://localhost/"); // array destructuring
const [err] = await doSomethingAsync(); // array destructuring
```

## Example usage

```js
const [fetchError, response] = await fetch.try('https://api.example.com/data');
if (fetchError) {
    console.log('Failed to fetch data:', fetchError);
    return;
}

const { value: data, isError } = await response.json.try();
if (parseResult.isError) {
    console.log('Failed to parse JSON');
    return;
}

console.log('Received data:', data);
```

## Working (Naive) implementation
```js
class Result {
    error = undefined;
    value = undefined;
    isError = false;

    // this will allow to use both object and array destructuring
    [Symbol.iterator] = function* () {
        yield this.error;
        yield this.value;
        yield this.isError;
    }
}

Function.prototype.try = function (...args) {
    const result = new Result();

    try {
        const promiseOrValue = this(...args);

        if (promiseOrValue instanceof Promise) {
            return promiseOrValue
              .then(value => {
                result.value = value;
                return result;
              })
              .catch(error => {
                result.error = error;
                result.isError = true;
                return result;
              });
        }

        result.value = promiseOrValue;
        return result;
    }
    catch (error) {
        result.error = error;
        result.isError = true;
        return result;
    }
};
```

## Precedents and Web Compatibility

This proposal aligns with existing practices where libraries provide similar functionality, such as try-catch wrappers or error handling utilities. Adding this method to Function.prototype introduces a standardized, built-in mechanism for handling errors more fluidly across synchronous and asynchronous code, enhancing JavaScript's capabilities without breaking existing web standards.

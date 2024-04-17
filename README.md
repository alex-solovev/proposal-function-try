# Function.prototype.try/tryCall for JavaScript

## Rationale
In JavaScript, error handling typically relies on try-catch blocks, which can lead to more verbose code, especially when dealing with asynchronous operations. By introducing `Function.prototype.try` and `Function.prototype.tryCall`, developers can manage errors in a structured and consistent format, similar to the approach seen in Go. This method allows errors to be treated as regular return values, which encourages more robust error handling practices and prevents errors from being overlooked. The introduction of Function.prototype.try would streamline error management, making the code cleaner and more maintainable.

## Description
The proposed `Function.prototype.try` and `Function.prototype.tryCall` methods encapsulate function execution within an implicit try-catch block, returning a `Result` object that contains the fields `value`, `error`, and an `catched` boolean flag. For asynchronous operations, the method returns a `Promise` that resolves to a `Result` object, ensuring that error handling remains consistent across both synchronous and asynchronous contexts. This feature is particularly beneficial in scenarios involving multiple asynchronous operations, as it allows each operation's errors to be handled distinctly and directly.

In order to work with object methods `Function.prototype.tryCall` can be used, the first argument is `thisArg` similar to `Functions`'s `call`, `bind` and `apply` methods. 
Thanks to Kevin Gibbons for [suggesting that](https://es.discourse.group/t/function-prototype-try-for-error-handling/2016/7).

```js
const result = JSON.parse.try("}"); // no destructuring
const { error, value, catched } = JSON.parse.try("}"); // object destructuring
const [err, json, isErr] = await fetch.try("http://localhost/"); // array destructuring
const [err] = await doSomethingAsync(); // array destructuring
const result = user.create.tryCall(user); // using `tryCall` to bind `this`
```

## Example usage

```js
const [fetchError, response] = await fetch.try('https://api.example.com/data');
if (fetchError) {
    console.log('Failed to fetch data:', fetchError);
    return;
}

const { value: data, catched } = await response.json.try();
if (catched) {
    console.log('Failed to parse JSON');
    return;
}

const reportGenerator = new ReportGenerator(data);
// for object methods `Function.prototype.tryCall` could be used similar to `Function.prototype.call`
console.log('Received report:', reportGenerator.generate.tryCall(reportGenerator));
```

## Working (naive) implementation
```js
class Result {
    error = undefined;
    value = undefined;
    catched = false;

    // this will allow to use both object and array destructuring
    [Symbol.iterator] = function* () {
        yield this.error;
        yield this.value;
        yield this.catched;
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
                result.catched = true;
                return result;
              });
        }

        result.value = promiseOrValue;
        return result;
    }
    catch (error) {
        result.error = error;
        result.catched = true;
        return result;
    }
};

Function.prototype.tryCall = function (thisArg, ...args) {
    const func = this;
    const result = new Result();

    try {
        const promiseOrValue = func.call(thisArg, ...args);

        if (promiseOrValue instanceof Promise) {
            return promiseOrValue
              .then(value => {
                result.value = value;
                return result;
              })
              .catch(error => {
                result.error = error;
                result.catched = true;
                return result;
              });
        }

        result.value = promiseOrValue;
        return result;
    }
    catch (error) {
        result.error = error;
        result.catched = true;
        return result;
    }
};
```

## Precedents and Web Compatibility

This proposal aligns with existing practices where libraries provide similar functionality, such as try-catch wrappers or error handling utilities. Adding this method to Function.prototype introduces a standardized, built-in mechanism for handling errors more fluidly across synchronous and asynchronous code, enhancing JavaScript's capabilities without breaking existing web standards.

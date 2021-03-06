Closures are very powerful mechanism in the JavaScript programming language. All members of an object in the JavaScript are public by default. However, closures mechanism provides to objects possibility to have private members, and not only that. In this tutorial we will learn what are closures and what are benefits of using them in the JavaScript code.

So, what is the closure? Douglas Crockford, author of the book *JavaScript: The Good Parts*, wrote an excellent definition of the closure, which says: 

>The closure is an inner function which always has access to the variables and parameters of its outer function, even the outer function has returned.

Let's consider the following code:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var increment = function(step) {
        
        currentValue += step; 
        console.log('currentValue = ' + currentValue);
    };
    
    return increment;
}

var incrementCounter = counter(0);

incrementCounter(1);
incrementCounter(2);
incrementCounter(3);
```
 In the JavaScript language, functions are objects, so they can be passed as arguments to other functions and they can be also returned from other functions and assigned to variables. In our example, we have created the `counter` function, which returns the `increment` function and assignes it to the `incrementCounter` variable, so that variable contains a reference to the *increment* function (the objects in the JavaScript are being copied by reference, not by value). That reference enables us to call the `increment` function outside of the `counter` function's scope, so in our case, we call it from the global scope. If you are unfamiliar with JavaScript scopes, please read [this](https://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/) great article before continuing with this tutorial. Beside that, the line `var incrementCounter = counter(0);` initializes the `currentValue` variable to zero.  
 
The line `incrementCounter(1);` actually invokes the `increment` function with paramether 1. Since that function is a closure and it still has access to the variables and parameters of its outer function, although we called it outside of its outer function, the `currentValue` will be increased by 1 and its value will be changed to 1. In the next function calls, its value will be increased by 2 and 3, respectively, so the output of the code will be:    

```
currentValue = 1
currentValue = 3
currentValue = 6
```

In the JavaScript, local variables of a function will be destroyed after the function returns, unless there is at least one reference on them. In our example, the `currentValue` is referenced in the `increment` function, which is referenced in the global scope. Therefore, the `currentValue` and `increment` will not be destroyed until the whole script execution is being terminated. That's why we can invoke the `increment` function from the global scope (using the reference to it, stored in the `incrementCounter`) after the `counter` function returned.

The `counter` function also can return more than one function, wrapped in a object. If we had `decrement` function and wanted to return it, as well, that could be accomplished with the following code:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var increment = function(step) {
        
        currentValue += step; 
        console.log('currentValue = ' + currentValue);
    };
    
    var decrement = function(step) {
    
        currentValue -= step;
        console.log('currentValue = ' + currentValue);
    }
    
    return {increment: increment,
            decrement: decrement};
}
```

The returned object has `increment` and `decrement` properties, which values are `increment` and `decrement` functions, respectively. In that way, we exposed those functions to the global scope, so we can call them from it:

```JavaScript

var myCounter = counter(0);

myCounter.increment(1);
myCounter.increment(2);
myCounter.decrement(1);
myCounter.increment(3);
myCounter.decrement(2);
```

For sure, when just one function is being returned, it can also be wrapped into an object, but it doesn't make so much sense. If more than one function are being returned, they must be wrapped into an object, like in the example above. We could also have functions inside of the `counter` functions which we didn't want to expose to the global (outer) scope. For instance, we could have a "private" function for logging the `currentValue`:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var logCurrentValue() {
        console.log('currentValue = ' + currentValue);
    }
    
    var increment = function(step) {
        
        currentValue += step; 
        logCurrentValue();
    };
    
    var decrement = function(step) {
    
        currentValue -= step;
        logCurrentValue();
    }
    
    return {increment: increment,
            decrement: decrement};
}
```

In this case, the `logCurrentValue` function cannot be accessed from the global scope, since it wasn't returned from the `counter` function. It can be used just in `increment` and `decrement` functions. Also, the `currentValue` and `initValue` variables are private for the `counter` function object.  So, that's how private members can emulated in the JavaScript.

Let's see how will this mechanism work if we create more than one function object. For instance, we will create `myCounter1` and `myCounter2` objects and use them in the following way:

``` JavaScript
var myCounter1 = counter(0);
var myCounter2 = counter(3);

myCounter.increment(1);
myCounter.increment(2);
myCounter.decrement(1);
myCounter.increment(3);
myCounter.decrement(2);
```

## Use Cases

Closures are mostly used in callbacks (timers, event handlers, ...) and modules.

### Timers

Let's consider the following code sample:

``` JavaScript
for (var i = 0; i < 5; i++) {

    console.log(i);    // 0 1 2 3 4 
}
```

The output of this code are numbers from 0 to 4, sequencially.

However, the output of the following code will be completely different, the number 5 will be logged 5 times:

``` JavaScript
for (var i = 0; i < 5; i++) {

    setTimeout(function() { 
    
        console.log(i); 
            
    }, 5000);     // 5 5 5 5 5
}
```

That is because the `function() {console.log(i)}` will be called 5s (5000ms) after the first loop iteration, then it will be called 5s after the second loop iteration and so on. That practically means that all 5 for loop iterations will be executed before the first call of the function, so the value of `i` will be 5 (after all iterations are being executed) and it will be logged 5 times.

In order to get the desired output, we will need to use the closure mechanism, so the code should look like this:

``` JavaScript
for (var i = 0; i < 5; i++) {

    setTimeout((function(i) { 
    
        console.log(i); 
            
    })(i), 5000);     // 0 1 2 3 4
}
```
In the first iteration the value of the variable `i` is zero. Therefore, we are setting timeout for the following function:

```JavaScript
function (i) {
    console.log(i);
}(0)
```
Therefore, after the timeout, `console.log(0)` will be executed. The same will happen for other values of the variable *i*, so, finally, values `0 1 2 3 4` will be logged in the console. 

### Event Handlers

Let's consider the following example: there is a button on an HTML page and we want to show users information about how many times the button was clicked. We could write the following code to implement this functionality:

``` HTML
<!DOCTYPE html>
<html>
<head>
    <title></title>
	<meta charset="utf-8" />
</head>
<body>
    <button id="test-button">Test</button>
    <script>
        var counter = 0;
        var element = document.getElementById('test-button');

        //event handler
        element.onclick = function () {

            counter++;
            alert('Number of clicks:' + counter);
        };
    </script>
</body>
</html>
```

This code works fine, but we had to define a global variable for counting user clicks on the button. That's not recommended way of writing code, since we use the `counter` variable just in the event handler, so why wouldn't we declare it inside the handler? However, if we just move declaration to the handler's scope, it will not work as expected, because every time the handler is called (i.e. every time the user clicks on the button), the `counter` will be set to zero. The solution is to use a closure, like in the following code:

``` HTML
<!DOCTYPE html>
<html>
<head>
    <title></title>
	<meta charset="utf-8" />
</head>
<body>
    <button id="test-button">Test</button>
    <script>
        var element = document.getElementById('test-button');

        //event handler
        element.onclick = (function outer () {

            var counter = 0;
            
            return function inner () {

                counter++;
                alert('Number of clicks: ' + counter);
            };            
        })();
    </script>
</body>
</html>
```
In this code, we have defined the `outer` function which returns the `inner` function. The `outer` function is immediately invoked and after its execution, the `inner` function is being assigned to the `onclick` handler. That means that whenever the user clicks on the button, the `inner` function will be called. Since it is a closure, it has access to the `outer` function scope, so it can easily change the value of the `counter` variable. For sure, function names could be omitted, I've wrote them just for purpose of better understanding the code above, so the `onclick` handler could look like this:

``` JavaScript
element.onclick = (function outer () {

    var counter = 0;
    
    return function() {

        counter++;
        alert('Number of clicks: ' + counter);
    };            
})();
```

### The Module Pattern

One of the most popular patterns in the Javacript, the module pattern, uses closures in its implementation. For instance, the `counter` module should look like this:

``` JavaScript
var counter = (function() {

    var currentValue = 0;

    var increment = function (step) {

        currentValue += step;
        console.log('currentValue = ' + currentValue);
    };

    var decrement = function (step) {

        currentValue -= step;
        console.log('currentValue = ' + currentValue);
    }

    return {
        increment: increment,
        decrement: decrement
    };
})();
```
The module pattern is used for singleton objects, so there can be just one instance of the `counter` module.

### Function Factory

``` JavaScript
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```

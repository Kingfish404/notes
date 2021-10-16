# currying

By: Jin Yu
Last Modified: 2021-10-5

柯里化(currying)，即把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

比如函数`f(x,y)`，currying后就变成了`h(x)(y)`，但最终得到的结果不变，只不过调用的方式发生了变化。

```javascript
// js currying 封装函数
function curry(func) {
    return function curried(...args) {
        // Function.length 属性指明函数的形参个数
        if (args.length >= func.length) {
            return func.apply(this, args);
        } else {
            return function (...args2) {
                return curried.apply(this, args.concat(args2));
            }
        }
    };
}

// 被currying的函数
function sum(a, b, c) {
    return a + b + c;
}

let curriedSum = curry(sum);

console.log(curriedSum(1, 2, 3)); // 6，仍然可以被正常调用
console.log(curriedSum(1)(2, 3)); // 6，对第一个参数的柯里化
console.log(curriedSum(1)(2)(3)); // 6，全柯里化
```

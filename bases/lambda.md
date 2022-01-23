# Lambda Calculus

$$
\lambda x . (x+1)
$$

## Theory

Lambda is a pure function that only combined with function's `input` and `output`

$\lambda$ to normal function.

$$
\lambda x . (x+1) \quad \hArr \quad f(x)=x+1 \quad \hArr \quad x+1=f(x)
$$

$\alpha - conversion$

$$
\lambda x.\lambda y.2*x+y \quad \hArr \quad \lambda y.\lambda x. 2*y+x
$$
  
$\beta - reduction$

$$
\lambda x.\lambda y. 2 * x + y \quad 3 \quad \rArr \quad \lambda y.2*3+y \quad \hArr \quad \lambda x.2*3 +x
$$

$\eta - reduction$

$$
if \quad g(x) \equiv f(x) \quad \forall x \\
\lambda x.g(x) \quad \hArr \quad \lambda x.f(x)
$$

## Code

Lambda base coding by `javascript`

```javascript
// factorial function
const fac = n => (n == 0 ? 1 : n * fac(n - 1))

// fac = H fac      .1
const H = f => n => (n == 0 ? 1 : n * f(n - 1))

// fac = Y H        .2
// Y H = H fac      .3=1+2
// Y H = H (Y H)    .4=3+2
const Y = f => (x => f(v => x(x)(v)))(x => f(v => x(x)(v)))

console.log({ fac_result: fac(10) })
console.log({ fac_result: H(fac)(10) })
console.log({ result: Y(H)(10) })
console.log({
    result:
        (f => (x => f(v => x(x)(v)))(x => f(v => x(x)(v))))
            (f => n => (n == 0 ? 1 : n * f(n - 1)))
            (10)
})
```

## REF

- [The Lambda Calculus - Computer Science, Columbia University](http://www.cs.columbia.edu/~sedwards/classes/2016/4115-spring/lambda.pdf)
- [A λ-calculus interpreter by Tadeu Zagallo](https://tadeuzagallo.com/blog/writing-a-lambda-calculus-interpreter-in-javascript/)
- [ES6函数与Lambda演算 | 小蘿蔔丁](https://juejin.cn/post/6844903549273391117)
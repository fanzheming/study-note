# JavaScript中的AOP

## 概念

`AOP （面向切面编程）`，缩写为Aspect Oriented Programming，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是JAVA 中Spring框架的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

## 意图

将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码。

## 实践

实现前置通知，后置通知，环绕通知

```js
/**
 * 给方法加入前置切片函数
 * @param {function} func 切片函数
 * @param  {any} args 切换函数的参数
 * @return  {function}  增强后的函数
 */
Function.prototype.__before = function(func,...args){
    if(typeof func != 'function') throw new Error('参数必须是函数')
    const __self = this
    return function(){
        func.apply(__self,args)
        return __self.apply(__self,arguments)
    }
}

let plus = function(a,b){
    console.log(`执行了原函数`)
    return a+b
}

const logger = function(funcName){
    console.log(`${funcName}:----log-----`)
}

plus = plus.__before(logger,'plus')
const result = plus(1,2)
console.log(result)

// --------------------------------------------------------------------------------------------------------

/**
 * 给方法加入后置切片函数
 * @param {function} func 切片函数
 * @param  {any} args 切换函数的参数
 * @return  {function}  增强后的函数
 */
Function.prototype.__after = function(func,...args){
    if(typeof func != 'function') throw new Error('参数必须是函数')
    const __self = this
    return function(){
        const res = __self.apply(__self,arguments)
        func.apply(__self,args)
        return res
    }
}

let plus = function(a,b){
    console.log(`执行了原函数`)
    return a+b
}

const logger = function(funcName){
    console.log(`${funcName}:----log-----`)
}

plus = plus.__after(logger,'plus')
const result = plus(1,2)
console.log(result)

// --------------------------------------------------------------------------------------------------------

/**
 * 切入点对象
 * 记录目标函数，为了不让目标函数在环绕通知时多次调用
 */
class JoinPoint {
    constructor(target,args){
        this.target = target
        this.args = args
        this.isInvoked = false
    }

    invoke(thiz){
        if(this.isInvoked) throw new Error('你不能多次调用目标函数')
        this.isInvoked = true
        return this.target.apply(thiz || this.target ,this.args)
    }
}


/**
 * 环绕通知
 * 原方法的执行需在环绕通知方法中执行
 * @param {function} func 环绕通知的函数
 * @return  {function}  增强后的函数
 */
Function.prototype.__around = function(func){
    if(typeof func != 'function') throw new Error('参数必须是函数')
    const __self = this
    return function(){
        const joinPoint = new JoinPoint(__self,arguments)
        return func.call(this,joinPoint)
    }
}

let test = function() {
    console.log(this)
    console.log(`测试函数`)
}

//回调中的参数joinPoint(切入点)对象, 在适当的时机执行JoinPoint对象的invoke函数，调用目标函数
test = test.__around(function(joinPoint){
    console.log(`--------开始--------`)
    joinPoint.invoke(this)
    console.log(`--------结束--------`)
})

test()
```
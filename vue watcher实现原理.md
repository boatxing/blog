# vue源码分析之watcher实现
## 用法
```
new Vue({
  el: '#watch-example',
  data: {
    question: '',
    a:{b: 1}
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.getAnswer()
    }
  }
}
```
watch对象的key可以是path，如："a.b"
## init watch  
源文件 core/instance/state.js   
initProps->initMethods->observe(data)->initComputed->initWatch
```
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```
## 创建监听
循环vm.$options.watch，给每个key创建一个监听
```
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```
核心代码：new Watcher(vm, expOrFn, cb, options)，expOrFn就是key
```
Vue.prototype.$watch = function (
  expOrFn: string | Function,
  cb: any,
  options?: Object
): Function {
  const vm: Component = this
  if (isPlainObject(cb)) {
    return createWatcher(vm, expOrFn, cb, options)
  }
  options = options || {}
  options.user = true
  const watcher = new Watcher(vm, expOrFn, cb, options)
  if (options.immediate) {
    cb.call(vm, watcher.value)
  }
  return function unwatchFn () {
    watcher.teardown()
  }
}
```
## Watcher对象怎么订阅状态变化通知
core/observer/watcher.js   
简化版watcher
```
/* @flow */
import Dep, { pushTarget, popTarget } from './dep'


/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */
export default class Watcher {
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ) {
    this.vm = vm
    vm._watchers.push(this)

    this.cb = cb
    this.id = ++uid // uid for batching
    //this.getter = parsePath(expOrFn)
    //this.get()
    //简化如下：
    pushTarget(this)
    vm._data[expOrFn] //触发Object get方法
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    this.getter.call(vm, vm)
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
      dep.addSub(this)
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    this.run()
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    this.cb.call(this.vm, value, oldValue)
  }
}
```
pushTarget，core/observer/dep.js   
this.getter.call(vm, vm) 可以简化为vm._data[expOrFn]，获取value的时候会触发属性的get function（属性在initWatch之前已经做了observer），get后该watcher会被add到属性对应的dep实例的subs队列。
```
Object.defineProperty(obj, key, {
  enumerable: true,
  configurable: true,
  get: function reactiveGetter () {
    const value = getter ? getter.call(obj) : val
    if (Dep.target) { //有watcher
      //订阅
      dep.depend()
    }
    return value
  },
  set: function reactiveSetter (newVal) {
    ...
  }
})
```
depend（dep.js），Dep.target就是当前watcher
```
depend () {
  if (Dep.target) { //pushTarget时设置
    Dep.target.addDep(this)
  }
}
```
addDep，watcher.js    
将该watcher添加到subs队列
```
/**
 * Add a dependency to this directive.
 */
addDep (dep: Dep) {
    dep.addSub(this)
}
```
## notify
改变状态的时候会触发Object的set方法
```
Object.defineProperty(obj, key, {
  enumerable: true,
  configurable: true,
  get: function reactiveGetter () {
    ...
  },
  set: function reactiveSetter (newVal) {
    ...
    dep.notify()
  }
})
```
dep的notify方法调用watcher的update方法，update调用run方法，run方法：this.cb.call(this.vm, value, oldValue)
```
notify () {
  // stabilize the subscriber list first
  const subs = this.subs.slice()
  for (let i = 0, l = subs.length; i < l; i++) {
    subs[i].update()
  }
}
```

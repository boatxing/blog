# vue监听数组变化
源代码：src/core/observer/array.js  
- 创建数组对象
```
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)
```
- 给arrayMethods定义如下方法，其他数组继承这个对象
```
const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
```
observer/index.js中相关代码
```
if (Array.isArray(value)) {
  const augment = hasProto
    ? protoAugment
    : copyAugment
  augment(value, arrayMethods, arrayKeys)
  this.observeArray(value)
} else {
  this.walk(value)
}
```
- 这些方法就是arrayProto的代理，调用这些方法的时候，先调用数组的方法（original），然后再做observe和notify
```
methodsToPatch.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator (...args) {
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
```

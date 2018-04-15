# vue observe解析
将data和props的每一个属性变成reactive property。如果属性有新的value，新的value也会被变成reactive。  
源代码core/observer/index.js
## observe 方法
为每一个value创建一个observer实例，此过程会一直递归下去，直到每个属性都有一个observer实例
```
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob: Observer | void
  //已经存在observer实例
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

## Observer对象
每个属性都会绑定一个Observer实例，为属性收集依赖，分发更新
### 数组怎处理
为了能够监听数组变化，每个数组都从arrayMethods继承，参考《vue 监听数组变化》

### 一般对象
```
/**
 * Walk through each property and convert them into
 * getter/setters. This method should only be called when
 * value type is Object.
 */
walk (obj: Object) {
  const keys = Object.keys(obj)
  for (let i = 0; i < keys.length; i++) {
    defineReactive(obj, keys[i], obj[keys[i]])
  }
}
```

## defineReactive
定义reactive property，get value时触发getter，set value时触发setter。
```
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  //收集依赖
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  //递归每个子元素
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) { //watcher
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      //value改变时，重新observe
      childOb = !shallow && observe(newVal)
      //通知变化
      dep.notify()
    }
  })
}
```
## set
增加新的属性时，需要调用set方法将该属性变成reactive，然后通知更新
如，使用说明文档：https://cn.vuejs.org/v2/guide/reactivity.html#%E5%A3%B0%E6%98%8E%E5%93%8D%E5%BA%94%E5%BC%8F%E5%B1%9E%E6%80%A7

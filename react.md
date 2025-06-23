# React

```react
npx create-react-app ${react-project-name}
```

## 工具

### lodash

对象排序工具

### classname

动态classname工具

## 常见操作

### useState在list中添加对象

![image-20250622184601729](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622184601729.png)

### 参数父传子

![image-20250622185309399](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622185309399.png)

父组件中给子组件放入的值，子组件可以通过props.children取到

![image-20250622185733251](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622185733251.png)

### 参数子传父

通过函数传递给父组件

![image-20250622190004292](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622190004292.png)

### json-server模拟接口服务

### 通过axios发送接口请求

### 配置@别名路径

![image-20250623214952320](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623214952320.png)

## 关键Hooks

### useState

返回值是一个数组，包含两个元素，定义一个变量，设置一个set方法

```react
const [newValue, setNewValue] = useState(initValue);
```

### useRef

获取DOM

```
1.使用useRef创建ref对象，并与JSX绑定
const inputRef = useRef(null); // 获取到DOM之前初始值为null
<input type="text" ref={inputRef} />

2.在DOM可用时，通过inputRef.current拿到DOM对象
console.log(inputRef.current) // 输出input这个
//绑定完了之后 DOM对象就是input的实例，可以操作其中的属性实现一些操作
```

### useContext

![image-20250622190917156](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622190917156.png)

### useEffect

在React组件中创建由渲染本身引起的操作，一渲染完毕就做个事

```react
useEffect(() => {}, []) // 想要做的事代表的函数， 可选 依赖项 数组为空时函数只执行一次

useEffect(() => {}) // 初始执行 + 组件更新
useEffect(() => {}, []) // 初始执行
useEffect(() => {}, [count]) // 初始执行 + 依赖项变化时（count变化时）
```

![image-20250622191634609](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622191634609.png)

#### 清除副作用

return一个清除函数

![image-20250622192153874](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622192153874.png)

### useMemo

在组件每次重新渲染时缓存计算的结果

![image-20250623215710111](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623215710111.png)

```react
useMemo(() => {}, [count1]) // 检测count1 发生变化时 执行函数
```

### React.memo

![image-20250623220431788](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623220431788.png)

props为空则不渲染，不变化也不渲染

引用类型（数组、函数）在重新渲染时会给一个新对象

### useCallback

在组件多次重新渲染时缓存函数

```react
useCallback(() => {}, []) // 缓存的函数， 依赖项（空则此函数永远不重新渲染，非空则根据此项变动重新渲染）
可配合memo使用(相当于memo传入一个不会变动的函数，所以不会因为父组件的渲染导致此函数更新)
```

### forwardRef

暴露DOM节点给父组件

![image-20250623222733100](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623222733100.png)

### 自定义hooks

![image-20250622192815234](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250622192815234.png)

## Redux

自定义的store示例发生变化时，可以通过store.subscribe({}) 执行操作

![image-20250623135009400](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623135009400.png)

![image-20250623135039313](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623135039313.png)

### Redux Toolkit和react-redux

```react
npx create-react-app react-redux-pro
npm i @reduxjs/toolkit react-redux

npm run start
```



![image-20250623170427940](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623170427940.png)

## Router

![image-20250623212033270](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623212033270.png)

![image-20250623212054105](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623212054105.png)

### 路径参数和请求参数

![image-20250623212711965](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623212711965.png)

![image-20250623212741420](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623212741420.png)

### 嵌套路由配置

![image-20250623212946191](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623212946191.png)

### 默认二级路由

![image-20250623213649229](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623213649229.png)

### 404路由

![image-20250623213838770](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623213838770.png)

### ReactRouter两种路由模式

![image-20250623214027670](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623214027670.png)

## Ant Design

![image-20250623214803537](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250623214803537.png)

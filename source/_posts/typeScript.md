
---
layout: post
title: "TypeScript 基本语法"
date: 2021-10-09 10:55
comments: true
tags: 
	 - typeScript
---

## 定义

- 是 JavaScript 的一个超集，提供最新的和不断发展的 JavaScript 特性。
- 解决大型项目的代码复杂性

<!--more-->

#### 与 JavaScript 区别

| typeScript                           | javaScript                   |
| ------------------------------------ | :--------------------------- |
| 强类型（支持静态或动态类型）         | 弱类型（没有静态类型选项）   |
| 在编译期间发现并纠正错误（静态语言） | 在运行时发现错误（动态语言） |
| 最终转换成 javascript 在浏览器运行   | 在浏览器里直接运行           |
| 增强代码的可读性与可维护性           | 可读性和可维护性较 ts 差     |

#### 工作流程

![image-20211019160851404](/public/assets/img/ts-typical-workflows.jpg)

#### 安装

```typescript
npm install -g typescript
```

#### 初始化

```typescript
tsc --init
```

#### 编译 ts 文件

```typescript
tsc xxx.ts
```

## 数据类型

#### 枚举

1、 数字枚举
~~~typescript
enum Response {
No, // No 值为 0
Yes, // Yes 值为 1，默认成员值从 0 开始递增，可以指定第一个成员值
}
function respond(recipient: string, message: Response): void {
// ...
}
respond("Princess Caroline", Response.Yes)
~~~
2、字符串枚举

```typescript
enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT',
}
```

3、 常量枚举

使用`const`关键字修饰的枚举，会使用内联语法，不会为枚举类型生成任何 JavaScript

```typescript
const enum Direction {
  NORTH,
  SOUTH,
  EAST,
  WEST,
}

let dir: Direction = Direction.NORTH

// 对应的ES5代码如下：
;('use strict')
var dir = 0 /* NORTH */
```

4、异构枚举

```typescript
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = 'YES',
}
```

#### Unknown 类型

```typescript
let value: unknown

value = true // OK
value = 42 // OK
value = 'Hello World' // OK
value = [] // OK
value = {} // OK
value = Math.random // OK
value = null // OK
value = undefined // OK
value = new TypeError() // OK
value = Symbol('type') // OK
```

**注意：**`unknown` 类型只能被赋值给 `any` 类型和 `unknown` 类型本身

#### Tuple 类型

单个变量中存储不同类型的值，工作方式类似数组

```typescript
let tupleType: [string, boolean]
tupleType = ['semlinker', true]
```

#### Void 类型

表示没有任何类型，当一个函数没有返回值时，通常会见到其返回值类型为 void

```typescript
// 声明函数返回值为void
function warnUser(): void {
  console.log('This is my warning message')
}
```

#### Never 类型

表示那些永远不存在的值的类型，如那些总是会抛出异常或根本就不会有返回值的函数表达式

```typescript
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
  throw new Error(message)
}

function infiniteLoop(): never {
  while (true) {}
}
```

## 断言

#### 类型断言

- **“尖括号”语法**

  ```typescript
  let someValue: any = 'this is a string'
  let strLength: number = (<string>someValue).length
  ```

- **as 语法**

  ```typescript
  let someValue: any = 'this is a string'
  let strLength: number = (someValue as string).length
  ```

#### 非空断言

- **忽略 undefined 和 null 类型**

  ```typescript
  function myFunc(maybeString: string | undefined | null) {
    const onlyString: string = maybeString // Error
    const ignoreUndefinedAndNull: string = maybeString! // Ok
  }
  ```

- **调用函数时忽略 undefined 类型**

  ```typescript
  type NumGenerator = () => number

  function myFunc(numGenerator: NumGenerator | undefined) {
    const num1 = numGenerator() // Error
    const num2 = numGenerator!() //OK
  }
  ```

- **确定赋值断言**

  ```typescript
  let x!: number // TypeScript编译器就会知道该属性会被明确地赋值
  initialize()
  console.log(2 * x) // Ok

  function initialize() {
    x = 10
  }
  ```

## 类型守卫

类型守卫就是缩小类型范围

#### in 关键字

```typescript
class Person {
  name = 'xiaomuzhu'
  age = 20
}

class Animal {
  name = 'petty'
  color = 'pink'
}

function getSometing(arg: Person | Animal) {
  if ('age' in arg) {
    console.log(arg.color) // Error
    console.log(arg.age) // ok
  }
  if ('color' in arg) {
    console.log(arg.age) // Error
    console.log(arg.color) // ok
  }
}
```

#### instanceof 关键字

```typescript
class Person {
  name = 'xiaomuzhu'
  age = 20
}

class Animal {
  name = 'petty'
  color = 'pink'
}

function getSometing(arg: Person | Animal) {
  // 类型细化为 Person
  if (arg instanceof Person) {
    console.log(arg.color) // Error，因为arg被细化为Person，而Person上不存在 color属性
    console.log(arg.age) // ok
  }
  // 类型细化为 Person
  if (arg instanceof Animal) {
    console.log(arg.age) // Error，因为arg被细化为Animal，而Animal上不存在 age 属性
    console.log(arg.color) // ok
  }
}
```

#### typeof 关键字

```typescript
function padLeft(value: string, padding: string | number) {
  if (typeof padding === 'number') {
    return Array(padding + 1).join(' ') + value
  }
  if (typeof padding === 'string') {
    return padding + value
  }
  throw new Error(`Expected string or number, got '${padding}'.`)
}
```

## 联合类型与类型别名

#### 联合类型

属性为多种数据类型之一，如字符串或数字，使用｜作为标记

```typescript
function formatCommandline(command: string[] | string) {
  let line = ''
  if (typeof command === 'string') {
    line = command.trim()
  } else {
    line = command.join(' ').trim()
  }
}
```

#### 类型别名

给一个类型起个新名字，类型别名和接口很像，但可以作为原始值、联合类型、元组及其他手写的类型

```typescript
type some = boolean | string

const b: some = true // ok
const c: some = 'hello' // ok
const d: some = 123 // 不能将类型“123”分配给类型“some”
```

类型别名可以是泛型:

```typescript
type Tree<T> = {
  value: T
  left: Tree<T>
  right: Tree<T>
}
```

**区别于 interface：**

- interface 只能用于定义对象数据类型
- type 的声明方式除了对象外可以定义为交叉、联合、原始类型等，类型声明的方式适用范围更广泛
- interface 方式可以实现接口的 extends 和 implements
- interface 可以实现接口合并声明

## 交叉类型

将多个类型合并为一个类型，包含了所需的所用类型，通过`&`运算符叠加类型

#### 同名基础类型属性的合并

```typescript
interface X {
  c: string
  d: string
}

interface Y {
  c: number
  e: string
}

type XY = X & Y
type YX = Y & X

let p: XY
let q: YX
```

混入后成员 c 的类型为 `string & number`，即成员 c 的类型既可以是 `string` 类型又可以是 `number` 类型。很明显这种类型是不存在的，所以混入后成员 c 的类型为 `never`

#### 同名非基础类型属性的合并

```typescript
interface D {
  d: boolean
}
interface E {
  e: string
}
interface F {
  f: number
}

interface A {
  x: D
}
interface B {
  x: E
}
interface C {
  x: F
}

type ABC = A & B & C

let abc: ABC = {
  x: {
    d: true,
    e: 'semlinker',
    f: 666,
  },
}

console.log('abc:', abc)
```

打印结果如下：

![image-20211019153648826](/Users/mlamp/Library/Application Support/typora-user-images/image-20211019153648826.png)

在混入多个类型时，存在相同成员，且成员类型为非基本数据类型，是可以成功合并的

## 函数

#### 可选参数、默认参数、剩余参数

```typescript
// 可选参数
function createUserId(name: string, id: number, age?: number): string {
  return name + id
}

// 默认参数
function createUserId(
  name: string = 'semlinker',
  id: number,
  age?: number
): string {
  return name + id
}

// 剩余参数
const result = (...rest: number[]) => {
  return [...rest].reduce((total, num) => total + num)
}
```

**注意：**如果有多个参数，可选参数是要放在普通参数的后面，剩余参数必须写在最后面，否则编译不通过报错,

#### 函数重载

使用相同名称和不同参数数量或类型创建多个方法的一种能力

```typescript
function add(a: number, b: number): number
function add(a: string, b: string): string
function add(a: string, b: number): string
function add(a: number, b: string): string
function add(a: Combinable, b: Combinable) {
  // type Combinable = string | number;
  if (typeof a === 'string' || typeof b === 'string') {
    return a.toString() + b.toString()
  }
  return a + b
}
```

## 泛型

允许同一个函数接受不同类型参数的一种模板，保留参数类型，创建可复用的组件。

```typescript
function returnItem<T>(para: T): T {
  return para
}
```

#### 多个类型参数

```typescript
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]]
}

swap([7, 'seven']) // ['seven', 7]
```

#### 泛型变量

```typescript
function getArrayLength<T>(arg: Array<T>) {
  console.log((arg as Array<any>).length) // ok
  return arg
}
```

#### 泛型接口

```typescript
interface ReturnItemFn<T> {
  (para: T): T
}

const returnItem: ReturnItemFn<number> = (para) => para
```

#### 泛型约束

```typescript
type Params = number | string

class Stack<T extends Params> {
  private arr: T[] = []
  public push(item: T) {
    this.arr.push(item)
  }
  public pop() {
    this.arr.pop()
  }
}

const stack1 = new Stack<number>()
const stack2 = new Stack<boolean>()
```

编译结果：

![image-20211025122906722](/Users/mlamp/Library/Application Support/typora-user-images/image-20211025122906722.png)

#### 泛型类型与索引类型

```typescript
function getValue<T extends object, U extends keyof T>(obj: T, key: U) {
  return obj[key] // ok
}
```

#### 多重类型进行泛型约束

```typescript
interface FirstInterface {
  doSomething(): number
}

interface SecondInterface {
  doSomethingElse(): string
}

class Demo<T extends FirstInterface & SecondInterface> {
  private genericProperty: T

  useT() {
    this.genericProperty.doSomething() // ok
    this.genericProperty.doSomethingElse() // ok
  }
}
```

#### 泛型工具类

- `typeof`操作符可以用来获取一个变量声明或对象类型

  ```typescript
  interface Person {
    name: string
    age: number
  }

  const sem: Person = { name: 'semlinker', age: 33 }
  type Sem = typeof sem // -> Person

  function toArray(x: number): Array<number> {
    return [x]
  }

  type Func = typeof toArray // -> (x: number) => number[]
  ```

- `keyof`可以获取某种类型的所有键，返回类型是联合类型

  ```typescript
  interface Person {
    name: string
    age: number
  }

  type K1 = keyof Person // "name" | "age"
  ```

- `in`用来遍历枚举类型

  ```typescript
  type Keys = 'a' | 'b' | 'c'
  type Obj = {
    [p in Keys]: any
  } // -> {a: any, b: any, c: any}
  ```

- `Partial<T>`将某个类型里的属性全部变为可选项`?`

  ```typescript
  type Partial<T> = { [P in keyof T]?: T[P] }
  ```

- `Exclude<T, U>`从 T 中排出可分配给 U 的元素

  ```typescript
  type Exclude<T, U> = T extends U ? never : T
  ```

- `Omit<T, K>`忽略 T 中的某些属性

  ```typescript
  type Omit<T, K> = Pick<T, Exclude<keyof T, K>>

  type Foo = Omit<{ name: string; age: number }, 'name'> // -> { age: number }
  ```

## 类

- 抽象类：做为其它派生类的基类使用,它们一般不会直接被实例化,不同于接口,抽象类可以包含成员的实现细节

  ```typescript
  abstract class Animal {
    abstract makeSound(): void
    move(): void {
      console.log('roaming the earch...')
    }
  }
  ```

- 访问限定符，`public`、`private`、`protected`

  > - 在 ts 类中，成员默认为 public，被此限定符修饰的成员可以被外部访问
  > - 成员被 private 修饰后，该成员只能在该类在内部访问
  > - 成员被 protected 修饰后，只可以在该类及该类的子类访问

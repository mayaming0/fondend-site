# ts数据类型

1. 原始类型：`number/string/boolean/null/undefined/symbol`
2. 对象类型：`object`（包括，数组、对象、函数等对象）
3. ts 新增类型：联合类型 / 自定义类型(类型别名) / 接口 / 元组 / 字面量类型 / 枚举 / void / any / 等等



## 原始类型

```typescript
// 数值类型
let age: number = 18

// 字符串类型
let myName: string = '小花'

// 布尔类型
let isLoading: boolean = false

// undefined
let un: undefined = undefined

// null
let timer:null = null

// symbol
let uniKey:symbol = Symbol() 
```

## 对象类型

#### 函数

```javascript
// function
// (普通函数)
function add(num1:number,num2:number):number {
    return num1 + num2
}
// （箭头函数）
const add2 = (num1:number,num2:number):number => {
    return num1 + num2
}
// （函数返回值是underfunded 或 无返回值）
function alert(msg:string):void{
    console.log(msg);
}
// （函数可选参数）
function slice(a?:number,b?:number):void{
    console.log(111);
}
// （函数默认参数）
function getName(firstName:string = 'mike',lastName?:string):string{
    return firstName + lastName
}
```

#### 对象

```javascript
// Object
let obj1 : {
    a:number,
    b:string,
    c?:Boolean
} = {
    a:1,
    b:'2'
}
```

#### 数组

```typescript
// Array
let list:number[] = [1,2,3]
let attr:Array<String> = ['1','2','3']
```



------



## 联合类型

```typescript
let 变量: 类型1 | 类型2 | 类型3 .... = 初始值
let arr1 :number | string = 1 // 可以写两个类型 
let timer:number|null = null
```



## 类型别名

```typescript
type 别名 = 类型
type s = string // 定义
----------------------------
const str1:s = 'abc'
const str2:string = 'abc' 

// 统一定义一个函数类型
type Fn = (n1:number,n2:number) => number 
const add3 : Fn = (a,b)=>{return a+b }

// 对象别名
type Person = {
    name:string,
    age:number,
    sayHi():void
}
let person:Person = {
    name:'li',
    age:88,
    sayHi(){
        console.log('hello');
    }
}
```



## 接口

当一个对象类型被多次使用时，有如下两种方式来来**描述对象**的类型，以达到复用的目的：

1. 类型别名，type
2. 接口，interface

##### 区别

- 相同点：都可以给对象指定类型
- 不同点:
  - 接口，只能为对象指定类型。**它可以继承。**
  - 类型别名，不仅可以为对象指定类型，实际上可以为**任意类型**指定别名



##### 属性接口

```typescript
interface Person {
  name: string;
  age: number;
}
function greet(person: Person) {
  console.log(`Hello, my name is ${person.name} and I'm ${person.age} years old.`);
}
let john: Person = {
  name: "John",
  age: 18
};
greet(john); // 输出：Hello, my name is John and I'm 18 years old.
```



##### 可选属性 与 只读属性 与 必选属性

```typescript
interface Person {
     readonly name: string;
     age?: number;
}
function greet(person: Person) {
     console.log(`my name is ${person.name}, my age is ${person.age}`);
}
let john: Person = {
     name: "John",
     age: 30,
};
john.name = "peter"; // 错误!
greet(john); // 输出：my name is John, my age is 30

// 必选
type IAnimal = {
    name?: string
    age?: number
    hobby?: string[]
}
type IDog = {
    [key in keyof IAnimal]-?: string
}
```



##### 函数接口

函数接口是一种描述函数类型的接口，用来规范函数的参数类型和返回值类型等信息。

```typescript
interface SayHi {
    (word:string):string
}
let sayHi:SayHi = function(word:string):string {
    return 'hello,'+word
}
console.log(sayHi('Mike'));// hello,Mike
```



##### 可索引接口

可索引接口是一种描述数组和对象的接口，用来规范它们的[索引类型](https://so.csdn.net/so/search?q=索引类型&spm=1001.2101.3001.7020)和索引值类型等信息。

```typescript
// 索引为数字的数组或对象
interface StringArray {
    [index: number]: string;
}
let colors: StringArray = ["Red", "Green", "Blue"];
let colors_2: StringArray = {0:'Red',1:'Green',2:'Blue'}
console.log(colors[0]); // 输出：Red

// 索引为字符串的对象
interface StringObject {
    [index: string]: string;
}
let words: StringObject = {
    'hello':'hello'
}
console.log(words['hello']); // 输出：hello
```



##### 类接口

类接口是一种描述类类型的接口，用来规范类的成员和属性等信息。

```typescript
interface Person {
  name: string;
  age: number;
  greet(name: string): void;
}

// implements关键字用于指定一个类或类接口应该实现另一个接口。
// 这里，我们使用implements关键字来指定Clock类应该实现ClockInterface接口
class Student implements Person {
  constructor(public name: string, public age: number) {}

  greet(name: string) {
    console.log(`Hello, ${name}, I'm ${this.name}.`);
  }
}

let john: Person = new Student("John", 18);
john.greet("Jane"); // 输出：Hello, Jane, I'm John.
```



##### 混合接口

混合接口是一种同时描述函数、类和对象等多种类型的接口，用来规范它们的成员和属性等信息

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}
  
function createCounter(): Counter {
    let counter = function(start: number) {} as Counter;
    // 或者 let counter = <Counter>function (start: number) { };
    counter.interval = 10;
    counter.reset = function() {};
    return counter;
}
    
let c = createCounter();
c(10);
c.reset();
c.interval = 5.0      
```



##### 接口继承

如果两个接口之间有相同的属性或方法，可以将**公共的属性或方法抽离出来，通过继承来实现复用** 语法

```typescript
interface IGoodItems2 extends IGoodItems{
  	price:number
}
```



## 元组

- **元组**是一种特殊的**数组**。有两点特殊之处

1. 它约定了的元素个数
2. 它约定了特定索引对应的数据类型

```typescript
// 例1
function useState(n: number): [number, (number)=>void] {
        const setN = (n1) => {
            n = n1
        }
        return [n, setN]
    }

const [num ,setNum] = useState(10) 

// 例2
var x: [string, number];
// 初始化他
x = ['hello', 10]; // 准确
```



## 字面量类型

字面量类型一般是配合联合类型一起使用的， 用来表示一组明确的可选值列表。

```typescript
// str1 是一个变量(let)，它的值可以是任意字符串，所以类型为:string
let str1 = 'hello TS'
// str2 是一个常量(const)，它的值不能变化只能是 ‘hello TS’，所以，它的类型为:‘hello TS’
const str2 = 'hello TS' 
```

```typescript
type Gender = 'girl' | 'boy'
// 声明一个类型，他的值 是 'girl' 或者是 'boy'
let g1: Gender = 'girl' // 正确
let g2: Gender = 'boy' // 正确
let g3: Gender = 'man' // 错误 
```



## 枚举

枚举（enum）的功能类似于**字面量类型+联合类型组合**的功能，来描述一个值，该值只能是 一组命名常量 中的一个。 

1. 一般枚举名称以大写字母开头
2. 枚举中的多个值之间通过 `,`（逗号）分隔
3. 定义好枚举后，直接使用枚举名称作为类型注解

```typescript
enum 枚举名 { 可取值1, 可取值2,.. }

// 使用格式：
枚举名.可取值 
```

枚举分为 **数值枚举** 和 **字符串枚举**。

- 数值枚举: 拥有自增长行为，默认为从 0 开始自增的数值；当然，也可以给枚举中的成员初始化值

  ```typescript
  enum Direction { Up = 10, Down, Left, Right }
  // console.log(Direction.Down)   => 11
  // console.log(Direction.Left)   => 12
  // console.log(Direction.Right)  => 13
  
  enum Direction { Up = 2, Down = 3, Left = 8, Right = 16 } 
  ```

- 字符串枚举:  字符串枚举没有自增长行为，因此，字符串枚举的每个成员必须有初始值

  ```typescript
  enum Direction {
    Up = 'UP',
    Down = 'DOWN',
    Left = 'LEFT',
    Right = 'RIGHT'
  } 
  ```



## any 类型

## 泛型

泛型指的是在定义函数/接口/类型时，**不预先指定具体的类型，而是在使用的时候在指定类型限制**的一种特性

### 泛型用法

#### 	1.在接口中使用泛型

```typescript
// 注意，这里写法是定义的方法哦
interface Search {
  <T,Y>(name:T,age:Y):T
}

let fn:Search = function <T, Y>(name: T, id:Y):T {
  console.log(name, id)
  return name;
}
fn<string,number>('li',11);//编译器会自动识别传入的参数，将传入的参数的类型认为是泛型指定的类型
```

#### 	2.在类中使用泛型

```typescript
class Animal<T> {
 name:T;
 constructor(name: T){
  this.name = name;
 }
 action<T>(say:T) {
   console.log(say)
 }
}
let cat = new Animal('cat');
cat.action('mimi')
```

#### 3.在函数中使用泛型

```typescript
function test <T> (arg:T):T{
  console.log(arg);
  return arg;
}
test<number>(111);// 返回值是number类型的 111
test<string | boolean>('hahaha')//返回值是string类型的 hahaha
test<string | boolean>(true);//返回值是布尔类型的 true
```



### 泛型约束

**为泛型添加约束来收缩类型(缩窄类型取值范围)**

#### 1 指定更加具体的类型  

```typescript
function id<Type>(value: Type[]): Type[] {
  console.log(value.length)
  return value
}
```

#### 2 添加约束

```typescript
// 例子1
interface Ilength {
    length:number
}
function idss<T extends Ilength>(arr:T):T {
    console.log(arr.length);
    return arr        
}
idss({ length:10, value:'00' })
// 例子2
type Animal<T extends hobbyList> = {
    name: string
    hobby: T
}
type hobbyList = {
    length: number
    push: (...args: any[]) => number
}
const animal: Animal<hobbyList> = {
    name: "阿黄",
    hobby: ['ball', 'flying-disc']
}
animal.hobby.push("run")
console.log(animal.hobby); //  [ 'ball', 'flying-disc', 'run' ]
// 例子3 联合类型
type Animal<T extends string | string[]> = {
    name: string
    hobby: T
}
const animal: Animal<string> = {
    name: "阿黄",
    hobby: "ball"
}
const animal2: Animal<string[]> = {
    name: "阿黄",
    hobby: ['ball', 'flying-disc']
}
// 例子4 交叉类型
type Animal<T extends arrayLength & arrayPush> = {
    name: string
    hobby: T
}
type arrayLength = {
    length: number
}
type arrayPush = {
    push: (...args: any[]) => number
}
type MyArray<T> = arrayLength & arrayPush & {
    forEach: (cb: (item: T, i: number, arr: MyArray<T>) => void) => void
}
const animal: Animal<MyArray<string>> = {
    name: "阿黄",
    hobby: ['ball', 'flying-disc']
}
animal.hobby.push("run")
console.log(animal.hobby.length); // 3
// 例子5 递归
type MenuItem = {
    label:string,
    url:string
}
type Menu<T> = {
    value:T
    children?:Menu<T>[]
}
const menu:Menu<MenuItem> = {
    value:{
        label:'home',
        url:'/home'
    },
    children:[{
        value:{
            label:'home',
            url:'/home'
        },
        children:[]
    }]
}
```

#### 3.多个类型变量

- 添加了第二个类型变量 Key，两个类型变量之间使用 , 逗号分隔。

- keyof 关键字接收一个对象类型，生成其键名称(可能是字符串或数字)的联合类型。
- 本示例中 keyof Type 实际上获取的是 person 对象所有键的联合类型，也就是：'name' | 'age'
- 类型变量 Key 受 Type 约束，可以理解为：Key 只能是 Type 所有键中的任意一个，或者说只能访问对象中存在的属性

```typescript
function getProp<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key]
}
let person = { name: 'xxx', age: 18}
getProp(person, 'name')
```



### 泛型-条件类型

#### 1.三元条件

```typescript
type IGetType<T> = T extends string ? 'str'
    : T extends number ? 'num'
    : T extends boolean ? 'bool'
    : 'other'
type INum = IGetType<number>// num
type IBool = IGetType<boolean>// bool
type IStr = IGetType<string>// str
type IOther = IGetType<unknown>// other
```



### 泛型工具类型

#### Partial< T >

Partial<T>的作用是将类型T中的所有属性变为可选属性

```typescript
interface IAnimal {
    name: string
    age: number
    color: string
}
type MyPartial = Partial<IAnimal>
// 等同于
type MyPartial = {
    name?: string;
    age?: number;
    color?: string;
}
// Partial 的实现
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```



#### Required< T >

Required与Partial相反，作用是将所有属性变成必选属性，用到的原理就是 '-?' 符号

```typescript
type MyRequired = Required<MyPartial>
// Required 的实现
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

#### Readonly< T >

将所有的属性换成只读

```typescript
type MyReadonly = Readonly<MyRequired>
// Readonly 的实现
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

#### Pick< T,K >

Pick的作用是在对象中选取一部分属性

```typescript
type MyPick = Pick<IAnimal, "name" | "age">
let animal: MyPick = {
    name: 'xxx',
    age: 18
}
```

#### Exclude< T, U >

Exclude<T, U>表示从T中排除U类型或类型集合

```typescript
// 下面代码排除了string类型
type MyExclude = Exclude<string | number | boolean, string>
```

#### Omit< T,K >

Omit<T, K>的含义是从T对象类型中删除K

```typescript
type OmitAnimal = Omit<IAnimal, "age" | "color">
// 实现
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

#### Record< K, T >

Record可以理解为批量创建一个对象中的属性，其中键值或其集合是K，类型是T

```typescript
type RecordAnimal = Record<"name" | "color", string> & Record<"age", number>
let record_animal:RecordAnimal = {
    name: 'xxx',
	color:'red',
	age:0
}
```

#### NonNullable< T >

NonNullable<T>一般用来从类型T中排除空类型，即判断是否存在null或undefined，如果不是则返回原类型

```typescript
NonNullable<null | undefined | string | boolean | number> // 表示 string | number | boolean
```

#### ReturnType< T >

ReturnType<T> 的作用是用于获取函数 T 的返回类型。

```typescript
type T1 = ReturnType<() => string> // string
type T2 = ReturnType<(s: string) => void> // void
```


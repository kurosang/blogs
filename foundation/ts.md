# typescript 查漏补缺

## 基础

1. ts 的文件无法直接运行，需要运行 tsc index.ts 将它转化为 index.js，在通过 node index.js 运行。（可以使用 ts-node 这个工具执行运行）

2. 我们定义一个变量的静态类型时，除了知道它是这个类型，还应该知道这个变量会具有这个类型下所有的属性和方法。

3. 基础类型和对象类型

```
// 基础类型 null, undefined, symbol, boolean, void
const count: number = 123;
const teacherName: string = 'Dell';

// 对象类型

class Person {}

const teacher: {
  name: string;
  age: number;
} = {
  name: 'Dell',
  age: 18
};

const numbers: number[] = [1, 2, 3];

const dell: Person = new Person();

const getTotal: () => number = () => {
  return 123;
};

```

4. 类型注解和类型推断

type annotation 类型注解, 我们来告诉 TS 变量是什么类型

type inference 类型推断, TS 会自动的去尝试分析变量的类型

如果 TS 能够自动分析变量类型，我们就什么也不需要做了

如果 TS 无法分析变量类型的话(推断为 any 时)，我们就需要使用类型注解

5. 函数相关类型

```
// 基本类型
function add(first: number, second: number): number {
  return first + second;
}
没有return的时候
function sayHello(): void {
  console.log('hello');
}
// never表示这个函数永远不可能执行到最后
function errorEmitter(): never {
  while(true) {}
}
```

6. 以上基础总结

```
// 基础类型, boolean, number, string, void, undfined, symbol, null
let count: number;
count = 123;

// 对象类型, {}, Class, function, []
const func = (str: string) => {
  return parseInt(str, 10);
};

const func1: (str: string) => number = str => {
  return parseInt(str, 10);
};

const date = new Date();

// 其他的 case
interface Person {
  name: 'string';
}
const rawData = '{"name": "dell"}';
const newData: Person = JSON.parse(rawData);

let temp: number | string = 123;
temp = '456';

```

7. 数组和元组

```
// 数组
const arr: (number | string)[] = [1, '2', 3]
const stringArr: string[] = ['a', 'b', 'c']
const undefinedArr: undefined[] = [undefined]

// type alias 类型别名
type User = { name: string; age: number }
const objectArr1: User[] = [{ name: '123', age: 2 }]

class Teacher {
  name: string
  age: number
}
// ts认为虽然第二项不是Teacher类new出来的，但是属性一致，也ok
const objectArr: Teacher[] = [
  new Teacher(),
  {
    name: 'dell',
    age: 28,
  },
]

// 元组 tuple
// 数组 (number | string)[] 的写法不能约束数组有多少项 并且 每一项具体是什么类型
const teacherInfo: [string, string, number] = ['Dell', 'male', 18]
// 例如：excel 或者 csv产生的数据结构可以用元组管理
const teacherList: [string, string, number][] = [
  ['dell', 'male', 19],
  ['sun', 'female', 26],
  ['jeny', 'female', 38],
]

```

8. interface 接口

类型别名 type 和 interface 接口有什么区别？基本无区别。type 可以直接等于一个基本类型比如 string，但是 interface 只能是对象或者函数。通用规范是能用接口就用接口，不行采用类型别名。

看一个案例：

```
interface Person {
  name: string;
  age?: number;
}

const getPersonName = (person: Person): void => {
  console.log(person.name);
};

const setPersonName = (person: Teacher, name: string): void => {
  person.name = name;
};

const person = {
  name: 'dell',
  sex: 'male'
};

// 1.getPersonName(person);
// 2.getPersonName({
  name: 'dell',
  sex: 'male'
})
```

上面，我们如果调用 1 ，ts 是不会报错的，因为它检查到这个变量里面有 name 字段，但如果是用对象字面量的方式传进去，ts 会进行强校验，发现有属性不一致就会报错。

**类** 应用 **接口** ：`interface Teacher extends Person`
**类** 应用 **接口** ：`class User implements Person`表示 User 类一定要有 Person 接口的属性和方法，没有就会报错。

interface 定义函数（还有索引的定义，具体看文档，比较少用）：

```
interface SayHi {
  (word: string): string
}

const say: SayHi = (word: string) => {
  return word
}
```

interface 只是 ts 帮助我们在开发时使用的东西，编译成 js 时会剔除掉，不会变成 js 代码。

9. 类的定义与继承

```
class Person {
  name = 'dell';
  getName() {
    return this.name;
  }
}

class Teacher extends Person {
  getTeacherName() {
    return 'Teacher';
  }
  getName() {
      // super表示父类
    return super.getName() + 'lee';
  }
}

const teacher = new Teacher();
console.log(teacher.getName());
console.log(teacher.getTeacherName());

```

10. 类中的访问类型和构造器

- private, protected, public 访问类型
- public 允许我在类的内外被调用
- private 允许在类内被使用
- protected 允许在类内及继承的子类中使用

**constructor**

```
class Person {
  // 传统写法
  // public name: string;
  // constructor(name: string) {
  //   this.name = name;
  // }
  // 简化写法
  constructor(public name: string) {}
}
const person = new Person('dell');
console.log(person.name);
```

new 一个类时，一定会自动调用类的 constructor 方法

11. 类里面的静态属性，Setter 和 Getter

```
getter and setter
class Person {
  constructor(private _name: string) {}
  get name() {
    return this._name + ' lee';
  }
  set name(name: string) {
    const realName = name.split(' ')[0];
    this._name = realName;
  }
}
const person = new Person('dell');
console.log(person.name);
person.name = 'dell lee';
console.log(person.name);
```

单例模式结合 ts：

```
class Demo {
  private static instance: Demo
  // 把 constructor 定义为private，就可以避免new创建
  private constructor(public name: string) {}
  // 自己设计一个获取实例的方法
  // static 定义的属性/方法 直接挂在class类上，不是实例上
  static getInstance() {
    if (!this.instance) {
      this.instance = new Demo('dell lee')
    }
    return this.instance
  }
}

const demo1 = Demo.getInstance()
const demo2 = Demo.getInstance()
console.log(demo1.name)
console.log(demo2.name)
```

12. 抽象类

抽象类只能定义变量名，不能定义具体，而且它只能被继承，不能 new。

```
abstract class Geom {
  width: number;
  getType() {
    return 'Gemo';
  }
  abstract getArea(): number;
}
class Circle extends Geom {
  getArea() {
    return 123;
  }
}
class Square {}
class Triangle {}
```

interface 也有类似抽象类的概念,把通用的东西抽出来：

```
interface Person {
  name: string;
}

interface Teacher extends Person {
  teachingAge: number;
}

interface Student extends Person {
  age: number;
}

interface Driver {
  name: string;
  age: number;
}

const teacher = {
  name: 'dell',
  teachingAge: 3
};

const student = {
  name: 'lee',
  age: 18
};

const getUserInfo = (user: Person) => {
  console.log(user.name);
};

getUserInfo(teacher);
getUserInfo(student);

```

## 高级

13.打包/配置

运行 tsc 命令，会自动打包，读取 tsconfig.json 的设置。

但如果是 tsc index.js。这种指定打包文件的，

tsconfig.json 配置项{}:

`include:["./demo.ts"]`，指定要编译的文件

`exclude:["./demo.ts"]`，指定不要编译的文件

compilerOptions 属性里的设置项：

`outDir`:重定向输出目录。

`rootDir`:打包读取路径

[更多](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

ts-node 会读取 tsconfig.json，他和 tsc 不是一个东西。

14.联合类型和类型保护

联合类型：就是 `|`

```
interface Bird {
  fly: boolean;
  sing: () => {}
}

interface Dog {
  fly: boolean;
  bark: () => {}
}

// 通过类型断言和代码逻辑，做类型保护
function trainAnimal(animal: Dog | Bird){
  if(animal.fly){
    (animal as Bird).sing()
  }else{
    (animal as Dog).bark()
  }
}

// in 语法来做类型保护
function trainAnimal2(animal: Dog | Bird){
  if('sing' in animal){
    animal.sing()
  }else{
    animal.bark();
  }
}

// typeof 来做类型保护
function add(first: string | number, second: string | number){
  if(typeof first === 'string' || typeof second === 'string'){
    return `${first}${second}`;
  }
  return first + second
}

// instanceof 来做类型保护
// 注意NumberObj要用class定义，因为interface不能调用instanceof
class NumberObj {
  count:number;
}

function add2(first: object | NumberObj, second: object | NumberObj){
  if(first instanceof NumberObj && second instanceof NumberObj){
    return first + second;
  }
  return 0;
}
```

15. enum 枚举

一般解决这种场景：

```
function getResult(status: number) {
  if (status === Status.OFFLINE) {
    return 'offline'
  } else if (status === Status.ONLINE) {
    return 'online'
  } else if (status === Status.DELETED) {
    return 'deleted'
  }
  return 'error'
}

const result = getResult(1)
console.log(result)
```

```
enum Status {
  OFFLINE,
  ONLINE = 2,
  DELETED,
}
```

输出：0 2 3

一般从 0 开始，但如果中间某个设置了值，则该值往下都按这个直递增

注意，枚举还可以反查，即 `Status.ONLINE` 输出为 2，`status[2]`输出为 ONLINE。

16. 函数泛型 generic

```
function join<T>(a: T, b: T) {
  return `${a}${b}`
}

join<number>(1, 1)
```

```
function map<F>(a: F[]) {
  return a
}

map(['123'])
```

以上写法可以达到我们要求 join 函数的两个参数必须同时为 number 或者 string

主要是针对声明函数时，不确定参数的类型

17. 类中的泛型以及泛型类型

很多时候用联合类型都可以达到泛型的效果。但是这会导致不灵活，还有可能写很长的一串。

```
class DataManager<T> {
  constructor(private data: T[]) {}
  getItem(idx: number): T {
    return this.data[idx]
  }
}

const data = new DataManager<number>([1])
data.getItem(0)
```

我们可以使用 interface 结合 extends 来限制 T 一定要具备某个属性

```
interface Item {
  name: string
}

class DataManager<T extends Item> {
  constructor(private data: T[]) {}
  getItem(idx: number): string {
    return this.data[idx].name
  }
}

const data = new DataManager([{ name: 'kuro' }])
data.getItem(0)
```

除了 interface，限制基本类型也可以

```
class DataManager<T extends number | string> {
  constructor(private data: T[]) {}
  getItem(idx: number): string | number {
    return this.data[idx]
  }
}

const data = new DataManager([])
data.getItem(0)
```

除了上面两种（函数泛型/类泛型），泛型还可以作为一个具体的类型注解

比如：

```
const func: <T>(params: T) => T = <T>(params: T) => {
  return params
}
```

好像这样写也可以？？

```
const func = <T>(params: T): T => {
  return params
}

func<number>(1)
```

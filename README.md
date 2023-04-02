# typescript-learn
# 类(class)
传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件，但对于熟悉使用面向对象方式的程序员来讲就有些棘手，因为他们用的是基于类的继承并且对象是由类构建出来的
## 类
```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```
## 继承
```typescript
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    // 派生类包含了一个构造函数，它必须调用super()，它会执行基类(Animal)的构造函数。 而且，在构造函数里访问this的属性之前，我们一定要调用super()
    constructor(name: string) { super(name); }
    // 重写父类的方法
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
// 即使tom被声明为Animal类型，但因为它的值是Horse，调用tom.move(34)时，它会调用Horse里重写的方法：
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);

// Slithering...
// Sammy the Python moved 5m.
// Galloping...
// Tommy the Palomino moved 34m.
```

## 公共，私有与受保护的修饰符
```typescript
class Animal {
    // 当成员被标记成private时，它就不能在声明它的类的外部访问。比如：
    private name: string;
    constructor(theName: string, type: string) { 
      this.name = theName; 
      this.type = type
    }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
    // protected修饰符与private修饰符的行为很相似，但有一点不同，protected成员在派生类中仍然可以访问
    protected type: string;
}
new Animal("Cat", '猫科').name; // 错误: 'name' 是私有的.

class Cat extends Animal {
     // 你可以使用readonly关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。
    readonly sex: string;
    constructor(name: string, sex: string) {
        super(name, '猫科')
        this.sex = sex
    }

    public getAnimalType() {
        return `Hello, my animal type is ${this.type}`
    }
}

const cat = new Cat('花花', '母'); // 错误: 'name' 是私有的.
console.log(Cat.getAnimalType());
console.log(Cat.type); // 错误
cat.sex = '公' // 错误! sex 是只读的.
```

# 接口（Interfaces）
## 接口初探
```typescript
interface LabelledValue {
  label: string;
  value?: string; // 可选属性
  readonly type: string; // 只读属性
  [propName: string]: any; // 额外的属性
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);

```

## 函数类型
```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

## 类类型
与C#或Java里接口的基本作用一样，TypeScript也能够用它来明确的强制一个类去符合某种契约。
```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    // 必须要有的属性
    currentTime: Date;
    // 必须要有的方法
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

## 继承接口
```typescript
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

## 泛型(generic)
软件工程中，我们不仅要创建一致的定义良好的API，同时也要考虑可重用性。 组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。

在像C#和Java这样的语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。
```typescript
function identity<T>(arg: T): T {
    // 因为这些类型变量代表的是任意类型，所以使用这个函数的人可能传入的是个数字，而数字是没有.length属性的
    // console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
// 方法一
let output = identity<string>("myString");  // type of output will be 'string'
// 方法二：利用了类型推论 – 即编译器会根据传入的参数自动地帮助我们确定T的类型：
let output = identity("myString");  // type of output will be 'string'
```
### 泛型约束
```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
loggingIdentity(3);  // Error, number doesn't have a .length property
```

### 泛型工具类型
```typescript
// 1.typeof
interface Person {
  name: string;
  age: number;
}
type Sem = typeof sem; // type Sem = Person
const lolo: Sem = { name: "lolo", age: 5 }

const Message = {
    name: "jimmy",
    age: 18
}
type message = typeof Message;
/*
 type message = {
    name: string;
    age: number;
}
*/

// 2.keyof
interface Person {
  name: string;
  age: number;
}

type K1 = keyof Person; // "name" | "age"
type K2 = keyof Person[]; // "length" | "toString" | "pop" | "push" | "concat" | "join" 
type K3 = keyof { [x: string]: Person };  // string | number

// 3.in
// in 用来遍历枚举类型：
type Keys = "a" | "b" | "c"

type Obj =  {
  [p in Keys]: any
} // -> { a: any, b: any, c: any }

// 4.infer
type Flatten<T> = T extends Array<infer U> ? U : T;

// Flatten<string> <=> string
// Flatten<Array<number>> <=> number
// 我们使用infer关键字声明性地引入了一个名为的新泛型类型变量infer U表示待推断的函数参数
// 整句的意思为：如果 T 能 赋值给 Array<infer U>,则结果是Array<infer U> 里的类型U，否则返回T.

// 5.extends
// 在泛型约束中使用类型参数
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.

```
### 内置的工具类型
为了方便开发者 TypeScript 内置了一些常用的工具类型，比如 Partial、Required、Readonly、Record 和 ReturnType 等。
```typescript
// Partial 将类型的属性变成可选
// type Partial<T> = {
//   [P in keyof T]?: T[P];
// };
interface UserInfo {
    id: string;
    name: string;
}
type NewUserInfo = Partial<UserInfo>;
// interface NewUserInfo {
//     id?: string;
//     name?: string;
// }

// Required 将类型的属性变成必选
interface Person {
    name?: string,
    age?: number,
    hobby?: string[]
}

const user: Required<Person> = {
    name: "树哥",
    age: 18,
    hobby: ["code"]
}

// Readonly<T>  作用是将某个类型所有属性变为只读属性，也就意味着这些属性不能被重新赋值。
interface Todo {
 title: string;
}

const todo: Readonly<Todo> = {
 title: "Delete inactive users"
};

todo.title = "Hello"; // Error: cannot reassign a readonly property


// Pick 从某个类型中挑出一些属性出来
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};

// Record<K extends keyof any, T> 的作用是将 K 中所有的属性的值转化为 T 类型。
interface PageInfo {
  title: string;
}

type Page = "home" | "about" | "contact";

const x: Record<Page, PageInfo> = {
  about: { title: "about" },
  contact: { title: "contact" },
  home: { title: "home" },
};

// ReturnType 用来得到一个函数的返回值类型
type Func = (value: number) => string;
const foo: ReturnType<Func> = "1";

// Exclude 作用是将某个类型中属于另一个的类型移除掉。
type T0 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // "c"
type T2 = Exclude<string | number | (() => void), Function>; // string | number

// Extract<T, U> 作用是从 T 中提取出 U。
type T0 = Extract<"a" | "b" | "c", "a" | "f">; // "a"
type T1 = Extract<string | number | (() => void), Function>; // () =>void

// Omit<T, K extends keyof any> 作用是使用 T 类型中除了 K 类型的所有属性，来构造一个新的类型。
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, "description">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};

// NonNullable<T> 的作用是用来过滤类型中的 null 及 undefined 类型。
type T0 = NonNullable<string | number | undefined>; // string | number
type T1 = NonNullable<string[] | null | undefined>; // string[]


```

### xx
```typescript
```
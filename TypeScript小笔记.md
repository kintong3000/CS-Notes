**TypeScript**

https://wangdoc.com/typescript/object

### **类型约束**

介绍：为了避免给字符串类型分配一个数值，则整个针对字符串相关的操作都是错误的

```ts
//1.类型推断，不建议使用
let str1 = "abc"; // 推断 str 为字符串类型
str1 = 10; //报错，不能将数值赋值给字符串类型

//2.类型注解，建议使用
let str2: string = "abc"; // 推断 str 为字符串类型
str2 = 10; //报错，不能将数值赋值给字符串类型

//3.类型断言
let numArr = [1, 2, 3];
const result = numArr.find((item) => item > 2) as number;
result * 5;
//ts比较严谨，若 result 为未定义的时候，则不能与 5 相乘，所以报错（在js中可以）
//所以加上 as number 来约束 result 的类型
```



### **基础类型和联合类型**

```ts
let v1: string = "abc";
let v2: number = 10;
let v3: boolean = true;
let nu: null = null;
let un: undefined = undefined;
//联合类型：v4 只能被分配 string 和 null 类型
let v4: string | null = null;
let v5: 1 | 2 | 3 = 2;
```



### **数组**

```ts
//数组的类型约束
let arr1: number[] = [1, 2, 3];
let arr2: Array<string> = ["a", "b", "c"];
```



### **元组**

功能：指定数组的元素个数和每个元素的类型

```ts
let t1: [number, string, number?] = [1, "a", 2];
//'?'为可选项，该位置可不填，但是填了之后也只能是对应的类型
```



### **枚举**

```ts
//定义
enum MyEnum {
 A,
 B,
 C,
}

//访问
console.log(MyEnum.A);
console.log(MyEnum[0]);

//输出
0 
"A" 
```



### **函数**

```ts
function MyFn1(a: number, b: string): string {
 return a + b;
}
//a 和 b 规定了约束了参数的类型
//():string 约束了返回值的类型。可以为 void 表示函数无返回值

function MyFn2(a: boolean, b = 10, c?: string, ...rest: number[]): boolean {
 return a;
}
//b 和 c 都为可选项
//b 不填则默认为 10，c 不填默认没有
//...rest 为剩余参数，且剩余值为一个数组结构，则可对其进行约束，为...rest: number[]
//注意，必选参数在左，可选参数在右，否则报错

//使用
const f: boolean = MyFn2(true, 20, "zx", 1, 2, 3, 4, 5);
console.log(f);

//输出
true 
```

### **类型别名**

```ts
//若很多参数都需要定义约束类型
let a: string | number = 10;
let b: string | number = 20;
let c: string | number = 30;

//则使用类型别名
type MyType = string | number;
let a: MyType = 10;
let b: MyType = 20;
let c: MyType = 30;
```



### 对象

对象类型的最简单声明方法，就是使用大括号表示对象，在大括号内部声明每个属性和方法的类型。属性的类型可以用分号结尾，也可以用逗号结尾。

```ts
const obj:{
  x:number;
  y:number;
} = { x: 1, y: 1 };

type MyObj = {
  x:number;
  y:number;
};

const o1:MyObj = { x: 1 }; // 报错
const o2:MyObj = { x: 1, y: 1};
```



### **接口**

```ts
//正常定义对象类型
const obj = {
 name: "xiaomin",
 age: 15,
};

//使用接口
interface obj {
 name: string;
 age: number;
};

//使用该接口来定义对象
const obj: obj = {
 name: "a",
 age: 10,
};
```

interface 与 type 的区别有下面几点。

（1）`type`能够表示非对象类型，而`interface`只能表示对象类型（包括数组、函数等）。

（2）`interface`可以继承其他类型，`type`不支持继承。



### **泛型**

功能：若一个函数是比较通用的函数，如下面的函数，想让他处理一组字符串、布尔类型、数值类型，则可以使用泛型

```ts
//对函数定义泛型
function myFn<T>(a: T, b: T, ...r: T[]): T[] {
 return [a, b, ...r];
}

//使用
//也可以 const arr = myFn(1, 2, 3, 12, 3); 因为 ts 支持类型推断
const arr: number[] = myFn<number>(1, 2, 3, 12, 3);
console.log(arr);

//输出
[1, 2, 3, 12, 3] 
```




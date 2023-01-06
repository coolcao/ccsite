---
title: Angular依赖注入实现原理解析
author: coolcao
date: 2022-12-15 00:24:25
tags: [Angular, 依赖注入, 前端]
categories:
  - 技术博客
  - 原创
---

Angular 的依赖注入的实现基于 Reflect Metadata。

<!-- more -->

Reflect Metadata 是 ES7 的一个提案，它主要用来在声明的时候添加和读取元数据。

TypeScript 在 1.5+版本已经支持它。

- `npm i reflect-metadata --save`
- 在 `tsconfig.json` 中配置 `emitDecoratorMetadata` 选项

## TypeScript 装饰器

TS 中装饰器（Decorators）为我们在类的声明及成员上通过元编程语法添加标注提供了一种方式。
要启用装饰器，以及 `reflect-metadata` 的支持，在 `tsconfig.json` 中做如下配置：

```json
{
  "compilerOptions": {
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "types": ["reflect-metadata"],
    "target": "ES5"
  }
}
```

我们先看一个简单的 ts 装饰器的例子：

```ts
const Log = function (target) {
  console.log(target);
};

@Log
class User {
  private name: string;
  constructor(name: string) {
    this.name = name;
  }
}

const u = new User("coolcao");
console.log(u);
```

转义后的 js：

```js
var Log = function (target) {
  console.log(target);
};
var User = /** @class */ (function () {
  function User(name) {
    this.name = name;
  }
  User = __decorate([Log, __metadata("design:paramtypes", [String])], User);
  return User;
})();
var u = new User("coolcao");
console.log(u);
```

> 这里省略了转译后装饰器`__decorate` 和 元数据 `__metadata` 的定义。

在 ts 中的装饰器，如果开启了 `"emitDecoratorMetadata": true` ，会生成一个 `design:paramtypes` 的元数据，记录的是该装饰器装饰的目标函数的参数列表（如果该装饰器用于函数上）。
例如上面例子中 `Log` 装饰器就是用于类 `User` 上（相当于用于 User 的构造器上），其参数类型列表就是 `[String]` 。

也就是说，如果一个装饰器用于类上，我们能通过元数据拿到这个类构造器的参数列表。

Angular 的依赖注入就是基于此实现的，基于 TS 的装饰器以及 `reflect-metadata` 实现。

下面我们可以通过一段简单的代码，来说明 Angular 中依赖注入的实现。

## 简易依赖注入

```ts
import "reflect-metadata";

// 定义一个装饰器
const Injectable = function () {
  return function (target) {};
};

class OtherService {
  a = 1;
}

@Injectable()
class TestService {
  // 构造器中列出需要依赖注入的类
  constructor(public readonly otherService: OtherService) {}
  testMethod() {
    console.log(this.otherService);
  }
}

const Factory = function (target) {
  // 获取所有注入的服务
  var providers = Reflect.getMetadata("design:paramtypes", target); // [OtherService]
  var args = providers.map(function (provider) {
    return new provider();
  });
  return new target(...args);
};

Factory(TestService).testMethod(); // 1
```

首先我们定义了一个 `Injectable` 的装饰器，这个装饰器其实并没做任何事，目的就是在将 TS 转译为 JS 时，将该装饰器装饰的类的构造器参数记录到元数据中。

通过使用装饰器 `@Injectable()` 标记类 `TestService` ，这时我们就会将 `TestService` 类构造器的参数记录到元数据 `design:paramtypes` 中。

`TestService` 的构造器中将依赖的 `OtherService` 作为参数传入，但并未在构造器中对 `OtherService` 做具体的实例化操作。

因为我们所有的实例化操作都是在 `Factory()` 中进行，在 `Factory` 中我们通过元数据`design:paramtypes`拿到 `TestService` 构造器中的参数类型，即 `OtherService` 这个类，然后我们使用 `new` 对其进行实例化，也就是得到了一个 `OtherService` 这个类的一个实例对象，将该实例对象作为参数， 去实例化 `TestService` ，如此，我们便完成了类 `TestService` 中对类 `OtherService` 的依赖注入。

## 总结

Angular 中的依赖注入，其实现的原理就是 TS 中装饰器，以及 Reflect Metadata，也即 `reflect-metadata` 库。
因此，在 `tsconfig.json` 中配置的编译选项 `emitDecoratorMetadata` 和 `experimentalDecorators` 必不可少。

`experimentalDecorators` 配置 TS 对于装饰器的支持。
`emitDecoratorMetadata` 配置装饰器的元数据支持。

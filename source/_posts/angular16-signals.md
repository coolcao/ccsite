---
title: Angular16新特性-Signals初体验
date: 2023-05-21 16:36:43
tags: [Angular, 前端]
categories:
- 技术博客
- 原创
---


Angular 16 为我们带来了 `Signals`，为 Angular 带来更细粒度的响应式能力。

熟悉 `Solid.js` 的朋友应该知道 `Signals` 的概念，意为“信号”，是 `Solid.js` 响应式的基础。Angular 16 将 Signals 添加进来，使得Angular在响应式方面更胜一筹。

<!-- more -->

## Angular Signals基础

### `signal()`
Angular 通过 `signal()` 函数定义一个Signal。 一个Signal便是一个getter函数，直接通过函数调用取值。

```ts
const counter = signal(0);
console.log(counter());
```

要修改Signal的值，可以通过三个函数： `set()`, `update()`, `mutate()`

- `set()`: 直接传入新值，新值覆盖旧值
- `update()`: 传入一个更新函数，入参是当前值，返回修改后的值
- `mutate()`: 传入一个更新函数，但没有返回值，直接修改入参的对象（一般用于数组和对象）

```ts
counter = signal(0);
// set
this.counter.set(1);
// update
this.counter.update(val => val + 1);
```

`signal()` 函数还可以传入一个equal函数，来指定判断相等的依据。

比如，对于一个用户，我们指定其id属性用来判断是否相等的依据。

```ts
interface IUser {
  id: string;
  name: string;
  age: number;
}

// 传入equal函数，指定判断用户是否相等的判断函数
user = signal({ id: '', name: '', age: 0 }, {
    equal: (a: IUser, b: IUser) => a.id == b.id,
});

updateUser() {
    this.user.set({ id: '1', name: 'coolcao', age: 23 });
    this.user.set({ id: '1', name: 'Jack', age: 30 });
}

```

如上，我们指定根据id判断是否相等，当执行完 `this.user.set({ id: '1', name: 'coolcao', age: 23 });`后，紧接着再执行 `this.user.set({ id: '1', name: 'Jack', age: 30 });`，此时由于id未发生变化，因此，第二个set并不能成功执行，最后的user是 `{ id: '1', name: 'coolcao', age: 23 }`。


### `computed()`

Angular使用 `computed()` 函数定义计算式 signal， 其结果是由其他 signal 计算而来，而且其会根据计算的signal变化而发生变化。

```ts
const num = signal(1);
const double = computed(() => num() * 2);
```

当num发生变化时，double也会随之发生变化，double永远是num的2倍。


### `effect()`
定义signal的副作用函数。
副作用是什么？就是signal发生变化时的副作用。

```ts
num = signal(0);
double = computed(() => this.num() * 2);

effect(() => {
    console.log(this.num());
    console.log(this.double());
    console.log('---');
});
```

每当num发生变化时，都会触发effect()函数，从而打印出num和double的值。

## Signals实例与其他方式实现对比

我们通过一个简单的TODO项目，来实验一下Signals以及与原来Angular方式的不同。

大致界面如下：
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/blog/20230731182651.png)

上半部分是一个输入表单，用来输入TODO的标题与详细信息。
下半部分是未完成与已完成的TODO列表。其中，点击未完成事项前的复选框可以将其选中，标记为已完成。

### Angular原来的方式

对应模板：
```html
<nz-divider></nz-divider>
<nz-space nzDirection="vertical">
  <input *nzSpaceItem nz-input placeholder="代办标题" [(ngModel)]="todo.title" />
  <nz-textarea-count *nzSpaceItem [nzMaxCharacterCount]="100">
    <textarea rows="4" placeholder="代办详细信息" name="comment" nz-input [(ngModel)]="todo.desc"></textarea>
  </nz-textarea-count>
  <button *nzSpaceItem (click)="addTodo()" nz-button>添加代办</button>
</nz-space>
<nz-divider></nz-divider>
<nz-space>
  <nz-list *nzSpaceItem nzBordered nzHeader="❌ 未完成" style="width: 400px;">
    <nz-list-item *ngFor="let todo of this.unDone()">
      <label nz-checkbox [(ngModel)]="todo.done">
        <span nz-typography class="todo-title"><mark>{{todo.title}}</mark></span>
      </label>
      {{todo.desc}}
    </nz-list-item>
  </nz-list>
  <nz-list *nzSpaceItem nzBordered nzHeader="✅ 已完成" style="width: 400px;">
    <nz-list-item *ngFor="let todo of this.done()" style="text-decoration:line-through;">
      <label nz-checkbox [(ngModel)]="todo.done">
        <span nz-typography class="todo-title"><mark>{{todo.title}}</mark></span>
      </label>
      {{todo.desc}}
    </nz-list-item>
  </nz-list>
</nz-space>
```

组件类：
```ts
export class TodoListComponent {
  todos: ITodo[] = [];
  todo = {
    title: '',
    desc: '',
  };
  addTodo() {
    if (!this.todo.title) {
        return;
    }
    this.todos.push({
      id: this.todos.length,
      done: false,
      ...this.todo,
    });
    this.todo = { title: '', desc: '' };
  }
  doneTodo(id: number) {
    this.todos[id].done = true;
  }
  done() {
    return this.todos.filter(todo => todo.done);
  }
  unDone() {
    return this.todos.filter(todo => !todo.done);
  }
}
```

我们直接定义一个todos数组，添加TODO就是直接push新项目到todos数组。
标记为已完成与未完成，直接使用ngModel双向绑定到具体每个todo的done属性上。

在已完成与未完成两个列表，通过两个函数`done()`和`unDone()`对所有todos进行过滤已完成和未完成两个列表。


### signals方式

html模板：
```html
<nz-divider></nz-divider>
<nz-space nzDirection="vertical">
  <input *nzSpaceItem nz-input placeholder="代办标题" [(ngModel)]="todo.title" />
  <nz-textarea-count *nzSpaceItem [nzMaxCharacterCount]="100">
    <textarea rows="4" placeholder="代办详细信息" name="comment" nz-input [(ngModel)]="todo.desc"></textarea>
  </nz-textarea-count>
  <button *nzSpaceItem (click)="addTodo()" nz-button>添加代办</button>
</nz-space>

<nz-divider></nz-divider>

<nz-space>
  <nz-list *nzSpaceItem nzBordered nzHeader="❌ 未完成" style="width: 400px;">
    <nz-list-item *ngFor="let todo of this.unDone()">
      <label nz-checkbox [(ngModel)]="todo.done" (ngModelChange)="doneTodo(todo.id)">
        <span nz-typography class="todo-title"><mark>{{todo.title}}</mark></span>
      </label>
      {{todo.desc}}
    </nz-list-item>
  </nz-list>
  <nz-list *nzSpaceItem nzBordered nzHeader="✅ 已完成" style="width: 400px;">
    <nz-list-item *ngFor="let todo of this.done()" style="text-decoration:line-through;">
      <label nz-checkbox [(ngModel)]="todo.done" (ngModelChange)="unDoneTodo(todo.id)">
        <span nz-typography class="todo-title"><mark>{{todo.title}}</mark></span>
      </label>
      {{todo.desc}}
    </nz-list-item>
  </nz-list>
</nz-space>
```

组件类：
```ts
export class TodoListComponent {
  todos = signal<ITodo[]>([]);
  unDone = computed(() => this.todos().filter(todo => !todo.done));
  done = computed(() => this.todos().filter(todo => todo.done));

  todo = {
    title: '',
    desc: '',
  };

  addTodo() {
    if (!this.todo.title) {
      return;
    }
    this.todos.mutate(todos => {
      todos.push({
        id: todos.length,
        done: false,
        ...this.todo,
      });
    });
    this.todo = { title: '', desc: '' };
  }

  doneTodo(id: number) {
    this.todos.mutate(todos => {
      todos[id].done = true;
      return todos;
    });
  }

  unDoneTodo(id: number) {
    this.todos.mutate(todos => {
      todos[id].done = false;
      return todos;
    });
  }

}
```

使用signals的方式，定义一个ITodo[]数组类型的Signal，添加代办时通过`.mutate()`函数插入新的代办事项。
标记已完成和未完成，也是通过`.mutate()`函数来实现。
然后通过`computed()`函数定义两个计算signal，`done`和`unDone`，即已完成和未完成代办的signal。

插入新代办与标记完成状态，都是对 `todos` 这个signal进行操作，而 `done` 和 `unDone` 两个signal是通过 `todos` 计算而来，是响应式的。

在html模板中，直接通过 `done()` 和 `unDone()` 来访问已完成和未完成代办。


### RxJs中BehaviorSubject实现

熟悉Angular的朋友可能会发现，如上Signals的方式，其实使用RxJS的`BehaviorSubject`也可以实现。

> RxJS 是响应式编程 `ReactiveX` 的JS实现，也是响应式编程的集大成者。Angular中已内置RxJS。

模板文件：
```html
<nz-divider></nz-divider>
<nz-space nzDirection="vertical">
  <input *nzSpaceItem nz-input placeholder="代办标题" [(ngModel)]="todo.title" />
  <nz-textarea-count *nzSpaceItem [nzMaxCharacterCount]="100">
    <textarea rows="4" placeholder="代办详细信息" name="comment" nz-input [(ngModel)]="todo.desc"></textarea>
  </nz-textarea-count>
  <button *nzSpaceItem (click)="addTodo()" nz-button>添加代办</button>
</nz-space>

<nz-divider></nz-divider>

<nz-space>
  <nz-list *nzSpaceItem nzBordered nzHeader="❌ 未完成" style="width: 400px;">
    <nz-list-item *ngFor="let todo of this.unDone$ | async">
      <label nz-checkbox [(ngModel)]="todo.done" (ngModelChange)="doneTodo(todo.id)">
        <span nz-typography class="todo-title"><mark>{{todo.title}}</mark></span>
      </label>
      {{todo.desc}}
    </nz-list-item>
  </nz-list>
  <nz-list *nzSpaceItem nzBordered nzHeader="✅ 已完成" style="width: 400px;">
    <nz-list-item *ngFor="let todo of this.done$ | async" style="text-decoration:line-through;">
      <label nz-checkbox [(ngModel)]="todo.done" (ngModelChange)="unDoneTodo(todo.id)">
        <span nz-typography class="todo-title"><mark>{{todo.title}}</mark></span>
      </label>
      {{todo.desc}}
    </nz-list-item>
  </nz-list>
</nz-space>
```

组件类：
```ts
export class TodoListComponent {
  todos$ = new BehaviorSubject<ITodo[]>([]);
  done$ = this.todos$.pipe(map(todos => todos.filter(todo => todo.done)));
  unDone$ = this.todos$.pipe(map(todos => todos.filter(todo => !todo.done)));

  todo = {
    title: '',
    desc: '',
  };

  addTodo() {
    if (!this.todo.title) {
        return;
    }
    const todos = this.todos$.value;
    todos.push({
      id: todos.length,
      done: false,
      ...this.todo,
    });
    this.todos$.next(todos);
    this.todo = { title: '', desc: '' };
  }

  doneTodo(id: number) {
    const todos = this.todos$.value;
    todos[id].done = true;
    this.todos$.next(todos);
  }

  unDoneTodo(id: number) {
    const todos = this.todos$.value;
    todos[id].done = false;
    this.todos$.next(todos);
  }
}
```

RxJS 通过定义BehaviorSubject实例，所有的更新操作都通过`.next()`方法将新的数据发射出去。
在模板中，通过`async`管道直接订阅异步流。`done$`和`unDone$`都是通过`todos$`流通过管道操作符组合而来成新的异步流。

> 在RxJS中，定义异步流，约定俗成命名都会在最后加`$`，表示这是一个流。


### Signals和RxJS

我们可以发现，在之前版本的Angular，也可通过内置的RxJS的BehaviorSubject来实现更细粒度的响应式。
那为什么Angular16会引入Signals呢？是为了替代RxJS么？ 

显然不是。这两者本就不是一个层面的东西。

Angular引入Signals更重要的是优化其变更检测机制，因为Angular利用zone.js来对浏览器进行猴子补丁，来实现其变更检测。

在Angular引入Signals后，未来Angular很可能就不再需要zone.js，但这并不意味着其替代RxJS，实际上，RxJS依然是Angular社区中最热门的响应式编程方案，因为RxJS依然很强大。

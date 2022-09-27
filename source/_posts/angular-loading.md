---
title: Angular使用RxJS实现全局loading
date: 2022-09-27 15:00:32
tags: [Angular]
categories:
  - 技术博客
  - 原创
---

我们在实现前端页面时，经常会遇到使用 http 加载远程数据的情况，为了友好的用户体验，一般在请求 http 远程数据时，都会用一个加载动画来减轻用户的等待焦虑。

<!--more-->

但如果工程比较大，页面比较多，我们在每一个组件中都设置一个 loading 变量，然后在每次 http 请求时设置 loading 的值，不免显得有点麻烦。

在 Angular 里，我们可以使用 RxJs 的 BehaviorSubject 来实现一个全局的 loading 发射器，配合 http 拦截器，实现全局的 loading。

首先我们定义一个`LoadingService`，用于设置全局`loading`状态。

```typescript
import { Injectable } from "@angular/core";
import { BehaviorSubject } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class LoadingService {
  public loading$: BehaviorSubject<boolean> = new BehaviorSubject<boolean>(
    false
  );
  private loadingMap: Map<string, number> = new Map<string, number>();

  constructor() {}

  setLoading(url: string, loading: boolean): void {
    if (!url) {
      throw new Error(
        "The request URL must be provided to the LoadingService.setLoading function"
      );
    }

    if (loading === true) {
      const count = this.loadingMap.get(url) || 0;
      this.loadingMap.set(url, count + 1);
      this.loading$.next(true);
    } else if (loading === false && this.loadingMap.has(url)) {
      const count = this.loadingMap.get(url);
      this.loadingMap.set(url, count! - 1);
      if (this.loadingMap.get(url) == 0) {
        this.loadingMap.delete(url);
      }
    }
    if (this.loadingMap.size === 0) {
      this.loading$.next(false);
    }
  }
}
```

这里，我们定义一个`BehaviorSubject<boolean>`类型的变量`loading$`，用以表示全局的`loading`发射器。`loadingMap`用以保存某个 url 下 loading 状态的个数（这里要保存 loading 状态的个数是因为，有些页面可能不止一个 http 请求，一个 loading 状态对应一个 http 请求，当所有的 http 请求都处理结束时，整个页面的 loading 才算结束）。
然后定义`setLoading(url: string, loading: boolean)`函数用以设置某个 url 下的 loading 状态。当设置为 true 时，则计数器加 1，反之则计数器减 1，当计数器为 0 时，使用发射器`loading$.next(false)`发送 false，表明整个页面的 loading 状态已结束。

然后，定义一个 http 拦截器，拦截 http 请求，请求发送前设置 loading 为 true，请求结束时，设置 loading 为 false。

```typescript
import { Injectable } from "@angular/core";
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
  HttpResponse,
} from "@angular/common/http";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";
import { LoadingService } from "../service/loading.service";

@Injectable()
export class LoadingInterceptor implements HttpInterceptor {
  constructor(private loadingService: LoadingService) {}

  intercept(
    request: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    this.loadingService.setLoading(request.url, true);
    return next.handle(request).pipe(
      map<HttpEvent<any>, any>((evt: HttpEvent<any>) => {
        if (evt instanceof HttpResponse) {
          this.loadingService.setLoading(request.url, false);
        }
        return evt;
      })
    );
  }
}
```

这样我们在每次发送 http 请求时，就会由拦截器自动拦截，并设置 loading 状态。我们只需要在组件中引用`LoadingService#loading$`即可收到其发送的状态值。

```html
<nz-spin [nzSpinning]="loading$|async"> ... </nz-spin>
```

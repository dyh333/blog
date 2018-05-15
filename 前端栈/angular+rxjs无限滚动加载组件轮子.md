## 前言 ##
昨天将之前收藏的一篇 [Angular 和 RxJS 实现的无限滚动加载](https://zhuanlan.zhihu.com/p/34712967)文章仔细看完，并动手实现了一遍代码。项目中也用到过类似的滚动加载功能，就当是给自己造个轮子用了。

因为原文章主要侧重于知识点的讲解和代码逻辑实现，造出来的组件并不具备通用性，所以自己在源码的基础上抽取出了一个通用性的infinite-scroll的抽象组件类，使其能适用于任何滚动加载型的应用场景。

## 代码 ##
先看抽象组件类：InfiniteScrollComponent

```
export abstract class InfiniteScrollComponent<T> {
    private cache = [];
    protected itemHeight: number;
    protected numberOfItems: number;
    protected isLoading = false;
    protected isEnding = false;

    get itemResults(): Observable<T> {
        return this.itemResults$;
    }

    /** core */
    // 手动决定加载第几页数据的流
    private pageByManual$ = ...
    // scroll 事件的流
    private pageByScroll$ = ...
    // resize 事件的流
    private pageByResize$ = ...
    // 合并上述三个页码流
    private pageToLoad$ = ...
    // 基于pageToLoad$流来创建一个新的流，它将包含无限滚动加载列表中的数据
    private itemResults$ = ...
    /** end core */    

    abstract appendData(page: number): Observable<T>;
}
```
核心代码部分基本照抄，只是将itemResults$中的http服务请求修改成调用一个抽象的appendData()函数。另外，为了屏蔽外界组件对rxjs流的困惑，增加了一个get itemResults()属性。最后增加了两个状态字段isLoading和isEnding，分别表示加载中和无更多数据，提高界面的友好性。

为了能够支持自定义的model，这里用了<T>泛型。
由于该类是一个抽象类，所以需要外部组件来继承并实现抽象方法：

```
export class InfiniteScrollListComponent extends InfiniteScrollComponent<string>
  implements OnInit {
  constructor(private infiniteScrollService: InfiniteScrollService) {
    super();
  }

  ngOnInit() {
    this.itemHeight = 40;
    this.numberOfItems = 10;
  }

  appendData(page: number): Observable<string> {
    return this.infiniteScrollService.loadByPage(page);
  }
}
```

在该类的ngOnInit()函数中，设置了每个item的高度（即每行的高度）和每页加载的条目数。然后实现了appendData()方法，在这个方法里就可以根据业务逻辑去获取到不同的数据了。

值得注意的是，appendData()方法的page参数是存储抽象类中的，其实这个page参数也是整个数据流的核心，无论是哪种事件流，最终都转化为了计算出当前page值。

至此，基本就完工了，视图页可以自由发挥了。比如：

```
<table>
  <tbody>
    <tr *ngFor="let item of itemResults|async" [style.height]="itemHeight + 'px'">
      <td>{{item.name}}</td>
    </tr>
  </tbody>
</table>
<div style="display: flex; justify-content: space-around;">
  <nz-spin *ngIf="!isEnding" [nzSize]="'small'" [nzSpinning]="isLoading" [nzTip]="'正在读取数据...'"></nz-spin>
  <span *ngIf="isEnding">没有更多数据了...</span>
</div>
```
项目源码已开源在[github](https://github.com/dyh333/infinite-scroll-list)上，欢迎issue和star。
        
## 参考 ##
[Infinite scroll in Angular an RxJS](https://blog.strongbrew.io/infinite-scroll-with-rxjs-and-angular2/)

[译. 使用 Angular 和 RxJS 实现的无限滚动加载](https://zhuanlan.zhihu.com/p/34712967)

[视口的宽高与滚动高度](http://harttle.land/2016/04/24/client-height-width.html)
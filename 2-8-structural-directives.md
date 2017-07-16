Structural Directives
=========

* structural directive主要用于修改元素的HTML布局，通常都会添加、删除或修改HTML元素。
* 将structural directive作用与host元素后，directive将会作用与host元素本身以及它的子元素。
* structural directive不需要使用`[]`，但必须带有`*`的前缀。
* 同一个host元素可以应用多个attribute directive，但只能应用一个structural directive。（没有类似ng1中directive的优先级的概念）

### `*`前缀语法糖

* 实质上只是语法糖，angular会将其转化为`<ng-template>`。angular会尝试`origin => template attribute form => element form`。
* `ngIf`的转换例子：
``` typescript
  // origin
  <div *ngIf="hero" >{{hero.name}}</div>
  //           | |
  //            V
  // element form
  <ng-template [ngIf]="hero">
    <div>{{hero.name}}</div>
  </ng-template>
```
* `ngSwitch`的转换例子：
``` typescript
  <div [ngSwitch]="hero?.emotion">
    <happy-hero    *ngSwitchCase="'happy'"    [hero]="hero"></happy-hero>
    <sad-hero      *ngSwitchCase="'sad'"      [hero]="hero"></sad-hero>
    <confused-hero *ngSwitchCase="'confused'" [hero]="hero"></confused-hero>
    <unknown-hero  *ngSwitchDefault           [hero]="hero"></unknown-hero>
  </div>
  //             | |
  //              V
  <div [ngSwitch]="hero?.emotion">
    <ng-template [ngSwitchCase]="'happy'">
      <happy-hero [hero]="hero"></happy-hero>
    </ng-template>
    <ng-template [ngSwitchCase]="'sad'">
      <sad-hero [hero]="hero"></sad-hero>
    </ng-template>
    <ng-template [ngSwitchCase]="'confused'">
      <confused-hero [hero]="hero"></confused-hero>
    </ng-template >
    <ng-template ngSwitchDefault>
      <unknown-hero [hero]="hero"></unknown-hero>
    </ng-template>
  </div>
```

### 绑定右侧的字符串语法糖

* 以`ngFor`的转换为例，所有跟ngFor无关的property都会转换到host元素上：
``` typescript
  <div *ngFor="let hero of heroes; let i=index; let odd=odd; trackBy: trackById" [class.odd]="odd">
    ({{i}}) {{hero.name}}
  </div>
  //           | |
  //            V
  <ng-template ngFor let-hero [ngForOf]="heroes" let-i="index" let-odd="odd" [ngForTrackBy]="trackById">
    <div [class.odd]="odd">({{i}}) {{hero.name}}</div>
  </ng-template>
```
  * `let`关键词会将紧随其后的变量定义为template input variable，如`let-hero`，`let-i`和`let-odd`。
  * 其中的`of`和`trackBy`会被转换成`ngForOf`和`ngForTrackBy`，这两个其实是NgFor这个directive的两个`@input` property。
  * 每轮循环都会有自己的context，而NgFor会负责管理这个context，在每轮循环中将变量赋值在context对象上。除了上面提到的几个变量，还有一个变量是`$implicit`，它指的是每轮循环的循环项本身。类似上面的`let-hero`没有被赋值，它默认就指向`$implicit`。
* template input variable的作用域仅限于循环体内。（而template reference variable则在整个模板都有效）

### `<ng-template>`与`<ng-container>`

* `<ng-template>`本身永远不会直接渲染到最终页面，它会被替换成一行HTML注释。
* 如果`<ng-template>`的子元素不含有任何structural directive，则子元素不会被渲染！
* `<ng-container>`通常用于需要在同一个host元素上应用多个structural directive的情形，这时可以将每个directive都作用于嵌套的`<ng-container>`上。
* `<ng-container>`同样不会被渲染到最终页面上，它的作用类似于JS里的`{}`，用来包裹。

### 自己写structural directive

* 与attribute directive类似，但需要注入`TemplateRef`和`ViewContainerRef`这两个service。
* `TemplateRef`用来获取“转换后”的`<ng-template>`中的内容。
* `ViewContainerRef`用来获取view container，`<ng-template>`的内容最终要插入其中，如`this.viewContainer.createEmbeddedView(this.templateRef);`

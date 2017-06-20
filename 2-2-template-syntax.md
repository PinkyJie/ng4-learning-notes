Template Syntax
=======

* 插值绑定：`{{xxx}}`，其中的`xxx`可以是component类的property，也可以是component类的方法调用，也可以是简单的表达式计算。
* 插值本质上是属性绑定，即`[property]="expression"`。

### Template Expression

* `[property]="expression"`
* 属性绑定表达式的变量上下文：除了来自component实例外，还可能是template input variable（`let abc`）或template reference variable（`#abc`），还有可能来自directive的context。它们的优先级是`template variable > directive context > componennt member`。
* 不应该有副作用，其中的表达式不支持一些特殊的JS运算符，如`++`、`=`、`-=`，不支持`;`和`,`，不支持`new`。
* 应该幂等。

### Template Statement

* `(event)="statement"`
* 含有副作用，用来更新应用的state，支持`=`、`;`、`,`，但依然不支持`++`、`+=`以及位运算符，不支持`new`。
* 上下文与上面的expression一致。

### HTML中attribute和property的区别

* attribute来自HTML，而property来自DOM。
* 很多attribute都有其对应的property，如`id`，但也有很多没有，如`colspan`。很多property并没有对应的attribute，如`textContent`。
* attribute用来初始化property的值，然后它们之间的联系就断了。property会变化，但attribute不会。比如input控件，输入新值时它的property变了，但使用`getAttribute('value')`得到的依然是初始化时的值。
* angular的绑定改变的是property的值，而不是attribute。

### 几种新绑定

* attribute绑定：`[attr.aria-label]="help"`，主要用于ARIA和SVG，因为它们没有对应的property。
* class绑定：`[class.special]="isSpecial"`。
* style绑定：`[style.color]="isSpecial ? 'red' : 'green'"`。可以使用dash-stle或camelCase，支持加单位，如`[style.font-size.%]="!isSpecial ? 150 : 50`。
* 这几个绑定都会用绑定的值**覆盖**已存在的值。

### `[property]`

* canonical form：`bind-target="xxx"`，从component到view element。
* **等号左边的永远是property的名字，而不是attribute。**如可以使用`<span [innerHTML]="title"></span>`，一个反例就是`<td colspan="{{1 + 1}}">Three-Four</td>`会报错，因为`colspan`不是property。（这也是为什么angular提供上面的attribute绑定的原因，使用`[attr.colspan]="1 + 1"`即可。）
* 如不加`[]`，则angular认为是在用字面的string来初始化这个property，与HTML的默认机制一致。
* 插值绑定其实是特殊形式的属性绑定，如果要绑定的值是string，则推荐插值，若是非string的类型，推荐属性绑定。
* 对于XSS，插值和属性绑定以不同的方式处理`<script>`，插值会显示而属性绑定不会，但两者都会以安全的方式显示。

### `(event)`

* canonical form: `on-target="xxx"`，从view element到component。
* 等号左边的是target发出的event，时间对象可以使用变量`$event`访问。如果target是DOM元素，则为DOM事件，若是自定义的组件，则可以使用angular的`EventEmitter`来发出自定义事件：
``` typescript
deleteRequest = new EventEmitter<Hero>();

delete() {
  this.deleteRequest.emit(this.hero);
}

<hero-detail (deleteRequest)="deleteHero($event)" [hero]="currentHero"></hero-detail>
```

### `[(...)]`

* canonical form: `bindon-target="xxx"`，双向绑定。
* 只是属性绑定和时间绑定的语法糖，以下两种方式等价：
``` typescript
<my-sizer [(size)]="fontSizePx"></my-sizer>
<my-sizer [size]="fontSizePx" (sizeChange)="fontSizePx=$event"></my-sizer>
```
* `[(aaa)]="xxx"`双向绑定用等号右边表达式`xxx`的值来初始化`aaa`属性，然后当`aaaChange`事件发生时，将事件对系那个本身赋值给`xxx`属性。

### 常用的attribute directive

* `ngClass`接受一个键值对对象，key为class名，value的真假来决定是否添加这个class名。与`[class.xxx]`的区别是`ngClass`通常用来一次处理多个class名。
* `ngStyle`也类似，用来同时处理多个style，key为样式名，value是样式值。
* `ngModel`，不使用`ngModel`的时候我们需要这么写：
``` html
<input [value]="currentHero.name" (input)="currentHero.name=$event.target.value" >
```
针对不同的元素，需要处理不同的property以及使用不同的方法从element中提取所需的value。这正是`ngModel`出现的原因，它对不同的元素的不同value和event进行封装，抽象成统一`ngModel`和`ngModelChange`。

### 常用的structural directive

* `ngIf`，与通过`[class.hidden]`和`[style.display]`等方式来隐藏元素不同，`ngIf`直接移除元素。
* `ngIf`常常用来防止不小心访问到空值的属性，如：
``` typescript
<div *ngIf="currentHero">Hello, {{currentHero.name}}</div>
<div *ngIf="nullHero">Hello, {{nullHero.name}}</div>
```
* `ngFor`的右边并不是常规的template expression，而是自己定义的语法。如`let hero of heros`会定义一个template input variable。
* 除了template input variable外，还提供一个`index`变量供使用。
* `ngFor`可以含有多个语句，用`;`隔开，如`*ngFor="let hero of heroes; let i=index"`。
* `trackBy: `接受一个函数，这个函数翻回一个列表中对象的属性，用来唯一标识这个对象，这样`ngFor`就可以重用DOM。

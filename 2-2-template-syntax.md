Template Syntax
=======

* 插值绑定：`{{xxx}}`，其中的`xxx`可以是组件类的property，也可以是组件类的方法调用，也可以是简单的表达式计算。
* 插值本质上是属性绑定，即`[property]="expression"`。

### Template Expression

* `[property]="expression"`
* 属性绑定表达式的变量上下文：除了来自组件实例外，还可能是template input variable（`let abc`）或template reference variable（`#abc`），还有可能来自directive的context。它们的优先级是`template variable > directive context > componennt member`。
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

* canonical form：`bind-target="xxx"`，从组件到view element。
* **等号左边的永远是property的名字，而不是attribute。**如可以使用`<span [innerHTML]="title"></span>`，一个反例就是`<td colspan="{{1 + 1}}">Three-Four</td>`会报错，因为`colspan`不是property。（这也是为什么angular提供上面的attribute绑定的原因，使用`[attr.colspan]="1 + 1"`即可。）
* 如不加`[]`，则angular认为是在用字面的string来初始化这个property，与HTML的默认机制一致。
* 插值绑定其实是特殊形式的属性绑定，如果要绑定的值是string，则推荐插值，若是非string的类型，推荐属性绑定。
* 对于XSS，插值和属性绑定以不同的方式处理`<script>`，插值会显示而属性绑定不会，但两者都会以安全的方式显示。

### `(event)`

* canonical form: `on-target="xxx"`，从view element到组件。
* 等号左边的是target发出的event，时间对象可以使用变量`$event`访问。如果target是DOM元素，则为DOM事件，若是自定义的组件，则可以使用angular的`EventEmitter`来发出自定义事件，`emit()`的参数即为传出的自定义事件对象：
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

* `ngClass`：
  * 接受一个键值对对象，key为class名，value的真假来决定是否添加这个class名。与`[class.xxx]`的区别是`ngClass`通常用来一次处理多个class名。
* `ngStyle`：
  * 与ngClass也类似，用来同时处理多个style，key为样式名，value是样式值。
* `ngModel`：
  * 不使用`ngModel`的时候，针对不同的元素，需要处理不同的property以及使用不同的方法从element中提取所需的value。我们需要这么写：
  ``` html
  <input [value]="currentHero.name" (input)="currentHero.name=$event.target.value" >
  ```
  * 这正是`ngModel`出现的原因，它对不同的元素的不同value和event进行封装，抽象成统一`ngModel`和`ngModelChange`。

### 常用的structural directive

* `ngIf`：
  * 与通过`[class.hidden]`和`[style.display]`等方式来隐藏元素不同，`ngIf`直接移除元素。
  * 常常用来防止不小心访问到空值的属性，如：
  ``` typescript
  <div *ngIf="currentHero">Hello, {{currentHero.name}}</div>
  <div *ngIf="nullHero">Hello, {{nullHero.name}}</div>
  ```
* `ngFor`：
  * 右边并不是常规的template expression，而是自己定义的语法。如`let hero of heros`会定义一个template input variable。
  * 除了template input variable外，还提供一个`index`变量供使用。
  * 可以含有多个语句，用`;`隔开，如`*ngFor="let hero of heroes; let i=index"`。
  * `trackBy`接受一个函数，这个函数返回一个列表中对象的属性，用来唯一标识这个对象，这样`ngFor`就可以重用DOM。
* `ngSwitch`：
  * 由三个directive组成：`ngSwitch`、`ngSwitchCase`、`ngSwitchDefault`。其中第一个是attribute directive，使用`[ngSwitch]`，剩下的两个才是structural directive，需要使用`*`。

### Template reference variables(`#var`)

* canonical form：`ref-var`。
* 可以代表HTML原生的元素，angular的组件甚至web component。
* 声明的变量可以在模板的任何地方引用，不要在同一个模板里定义两个同名的变量。
* 默认情况下变量指向声明时所在的元素，但directive可以改变该变量的行为。一个例子：`ngForm`里`#myForm="ngForm"`，这时变量`myForm`不再指向HTML原生的form元素，而是指向angular的NgForm directive。

### 绑定的source和target

* 等号左边是target，右边是source。
* 左边通常都包裹在`[] () [()]`，右边通常是`"" {{}}`。
* directive的属性或方法都可以直接当做target，但要成为source必须定义为inputs和outputs。

### `@Input`和`@Output`

* 定义：
``` typescript
<hero-detail [hero]="currentHero" (deleteRequest)="deleteHero($event)"></hero-detail>

// define in class
@Input()  hero: Hero;
@Output() deleteRequest = new EventEmitter<Hero>();

// define in decorator
@Component({
  inputs: ['hero'],
  outputs: ['deleteRequest'],
})
```
* input用来接收数据（通常来自父组件），而output用来将组件的事件传递出去（通常直接传`EventEmitter`）。
* “输入/输出”是从target directive的角度来讲的。
* input/output可以声明别名：
``` typescript
// `myClick` is an alias, `clicks` is the name used internally
@Output('myClick') clicks = new EventEmitter<string>();

// define alias in decorator
@Directive({
  outputs: ['clicks:myClick']  // propertyName:alias
})
```

### Template expression operator

* 管道`|`：
  * 类似ng1的filter，就是一个函数，参数为原始值，返回转换后的值。
  * 可以chain多个`xxx | pipe1 | pipe2`。
  * 可以接受参数，`birthdate | date:'longDate'`。

* 安全访问属性`?.`：
  * 防止模板中访问空对象的属性导致报错：`{{nullHero?.name}}`。
  * 也可以使用其他方案，如`*ngIf="nullHero"`或`{{nullHero && nullHero.name}}`，但使用`?.`的好处就是如果有要访问对象很深的路径时，写起来比较方便，如`{{a?.b?.c?.d}}`。

* assert非空值`!`：
  * 主要是为Typescript的static null checking服务，如果开启了这个特性`--strictNullChecks`，则angular的模板转换为Typescript后，Typescript会检查模板里的值是否可能为null，是的话就会报错。但当我们明确知道某些情况下它不可能是null时（比如前面已经用`*ngIf`做过判断了），这时可以使用`!`来显式的告诉Typescript编译器不要报错，如：
  ``` typescirpt
  <div *ngIf="hero">The hero's name is {{hero!.name}}</div>
  ```

Lifecycle Hooks
=========

* 组件可以通过实现接口的方式来接入生命周期的不同阶段。这些接口通常含有同名的方法，如`OnInit`接口含有一个`ngOnInit()`的方法。
* directive也有声明周期，组件的生命周期里除去那些content和view特有的，其他的directive也有。
* “实现接口”并不是必须的，因为JS中并不存在接口的概念。angular只要发现组件中有相应方法就会在相应的声明周期进行调用。

### 所有的声明周期

* `ngOnChanges()`：
  * 每次`@Input`的property值改变时调用，在`ngOnInit()`前调用。
  * 感觉类似于React的`componentWillReceiveProps`。
  * 这里property的改变是指“值”的改变，如果是一个object，那么它内部属性的改变并不会导致`ngOnChange()`的调用，因为object的reference并没有改变。
* `ngOnInit()`：
  * 在第一次`ngOnChanges()`后调用，只会调用一次。
  * `contructor()`的逻辑应该尽量保持简单（如简单的赋值），其他复杂的初始化逻辑应该放在`ngOnInit()`中，如调用API获取值。
  * 在`contructor()`中是无法拿到`@Input`的property的值的，所以如果需要根据这个值来决定不同的初始化逻辑，那么需要在`ngOnInit()`中进行判断。(当然，在`ngOnChanges()`中也可以拿到`@Input`的值。)
* `ngDoCheck()`：
  * 每次change detection都会调用，第一次紧随`ngOnChanges()`和`ngOnInit()`之后。
  * 一旦有异步事件发生就会调用，逻辑需尽量保持轻。
* `ngAfterContentInit()`：
  * 在angular将外部内容（类似于ng1的transclusion，父组件模板中使用`<ng-content>`作为占位符，使用`<parent-comp><child-comp></child-comp></parent-comp>`将子组件的内容插入到父组件的占位符中，称之为"project"）插入到组件的view时调用。
  * 只调用一次，在第一次`ngDoCheck()`之后调用。
  * component特有。
* `ngAfterContentChecked()`：
  * 第一次调用紧随`ngAfterContentInit()`，后面每次`ngDoCheck()`之后调用。
  * component特有。
* `ngAfterViewInit()`：
  * 初始化组件的view和子view时调用，只调用一次，紧随第一次`ngAfterContentChecked()`之后。
  * component特有。
* `ngAfterViewChecked()`：
  * 第一次调用紧随`ngAfterViewInit()`，后面每次`ngAfterContentChecked()`之后调用。
  * 如若子view的property改变触发了此次调用，那么拿到的子view的property值是改变前的。
  * 在此回调中不能改变view中property的值，因为此时的生命周期初期已经checked的时期了，angular的undirectional data flow不允许在此回调立即改变已经checked的值。如果需要改变，必须使用setTimeout，否则angular会报错。
  * component特有。
* `ngOnDestroy()`：
  * angular销毁组件时调用。
  * 通常用来取消订阅事件、Observable，避免内存泄露。

### AfterView 与 AfterContent 的区别

* AfterView关注的是`@ViewChild`，也就是说子组件的tag直接出现在了父组件的模板中，父组件可以通过`@ViewChild`来获取子组件。
* AfterContent关注的是`@ContentChild`，子组件的tag并没有直接出现在父组件的模板中，只是以`<ng-content>`占位符的方式插入，父组件通过`@ContentChild`来获取子组件。
* `ngAfterViewChecked`需要遵循单向数据流的限制，即不允许在该回调中更改property的值。
* `ngAfterContentChecked`则没有单项数据流的限制。因为AfterContent是在AfterView之前调用的？


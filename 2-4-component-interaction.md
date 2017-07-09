Component Interaction
=========

* 父组件给子组件传递数据。
  * 通过`@Input` property。
  * `@Input`property可以自带setter，如
  ``` typescript
  private _name = '';

  @Input()
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }

  get name(): string { return this._name; }
  ```
  * 如果子组件需要基于多个`@Input`的变化来执行某操作，可以通过`ngOnChanges()`来实现。
* 子组件向父组件传递事件。
  * 通过`@Output` property。
  * `@Output` property本身是一个`EventEmitter`，子组件在自身的方法中调用`EventEmitter.emit(data)`，而父组件可以通过`@Output`右边的绑定中使用`$event`变量来访问传出的数据。
* 父组件通过template reference variable直接访问字组件的property和方法。
  * 只适用于在父组件模板中使用。
* 父组件使用`@ViewChild`来访问字组件的property和方法。
* 父子组件通过一个service来共享数据，service可以声明在父组件的`providers`中，只有父组件和字组件对此service可见。

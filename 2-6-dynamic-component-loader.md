Dynamic Component Loader
=========

* 动态加载组件：将一个组件动态的插入到父模板中。
* 流程：
  1. 在父模板中定义一个directive，作为插入点，这个directive需要注入`ViewContainerRef`，通过它获取container的元素引用，如`<ng-template insert-point>`。
  2. 父组件中注入`ComponentFactoryResolver`，通过它动态的创建组件类，它的参数就是需要动态创建的组件的类名，如
  ``` typescript
    let componentFactory = this.componentFactoryResolver.
      resolveComponentFactory(ComponentClass);
    let componentRef = viewContainerRef.createComponent(componentFactory);
  ```
  3. 通过返回的组件的ref即可访问组件的`@Input`property，如
  ``` typescript
    (<ComponentClass>componentRef.instance).data = xxx;
  ```

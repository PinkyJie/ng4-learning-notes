Attribute Directives
=========

* attribute directive主要用于修改元素的外观和行为。

### 定义和使用

* 一个attribute directive就是定义一个controller类，加一个`@Directive`的decorator，在decorator中声明这个directive对应的CSS `selector`（通常直接使用属性选择器`[attributeName]`）。
* 通常directive的controller类都会在构造函数中注入`ElementRef` service用来访问DOM元素（通过`ElementRef.nativeElement`来获取原生的DOM元素）。
* 必须将该directive的类引用加入`@NgModule`的metadata中去，放在`declarations`对应的数组里，这样才能将其暴露供其他组件使用。否则angular会直接报错，无法正常识别template中出现的未知directive。

### 响应事件

* 通过定义被decorator `@HostListener`修饰的event listner，即可让directive响应事件。如
``` typescript
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight('yellow');
  }
```
* 当然也可以自行在元素上绑定事件，使用`HostListener`的好处是它可以在directive销毁时自动销毁事件的绑定。

### 接受外部传值

* 通过定义被decorator `@Input`修饰的property，即可让directive接受外部传值。
* 通常`@Input`的property的名字与选择器名一致，这样就可以使用同一个名字来应用directive的同时并传值。
* 如果directive的选择器名不是一个好的传值变量名，则可以使用`@Input`的别名机制。
* 一个directive可以定义多个`@Input`的property用来接受多个传值。

### 什么时候需要`@Input`

* 当一个property出现在绑定`=`的右侧时，表明这个property属于该template对应的组件，组件和自身的template是互相信任的，这个property也是私有的，此时不需要`@Input`。
* 当一个property出现在绑定`=`的左侧时，表明这个property不属于该template对应的组件，所以这个property本身必须使用`@Input`来修饰，将其变成public的。

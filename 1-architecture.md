Architecture
==============

### Modules

* 每个Angular应用都包含一个root module，一般叫做`AppModule`，除此之外还可能包括很多feature modules。
* Anglar modules由decorator `@NgModule`和类组成，decorator是一个函数，一般接收一个metadata对象，作用就是将metadata与类进行关联。
* `@NgModule`的metadata包含一些重要属性：
  * `declarations`：这个module中包含的view classes，有3种：components、directives、pipes。
  * `exports`：上面`declarations`的子集，用于将module里的类暴露给外部module的组件模板使用。
  * `imports`：本module中定义的一个组件模板中若需要使用外部module的类，需要在这里声明。
  * `providers`：这个module中定义的各种用于创建service的类，全局可用。
  * `bootstrap`：root component，应用的主view，只有root module才有这个属性。
* 定义了root module以后就可以在`main.ts`中bootstrap了：

```
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

platformBrowserDynamic().bootstrapModule(AppModule);
```

* Angular自身带了很多library module，都带有`@angular`前缀。
* Angular module与Javascript自身的module的区别：
  * JS自身module一般一个文件就是一个module，通过`import`和`export`来引入和暴露类。
  * 而Angular的module一般都包括多个文件，定义时需要通过“JS module”先将类引入，然后把引入的类传入metadata实现与Angular module关联。

### Components

* Component控制屏幕上的某一块view区域，Component类通过其属性和方法来与view进行交互。
* Component也是由一个普通的类与decorator `@Component`组成的，其中一些常见的metadata属性有：
  * `selector`：在父模板中的CSS选择器。
  * `templateUrl`或`template`：模板URL或行内模板。
  * `providers`：一个数组，实现依赖注入的关键，声明本组件需要用到的service。

### 数据绑定

* 4种数据绑定：
  * `{{value}}`：插值，从组件到DOM。
  * `[property] = "value"`：属性绑定，把父类组件的value的值传入子类组件的property属性中去。
  * `(click) = "handler"`：事件绑定，点击时间发生时调用组件的handler方法。
  * `[(ng-model)] = "property"`：双向绑定。
* Angular在每次Javascript事件循环中处理一次所有的数据绑定，从根组件开始到左右的字组件。

### Directives

* Directive为Angular渲染模板提供一些指示。
* Directive同样是由普通的类与decorator `@Directive`组成的，Component也是一中特殊的Directive——带有模板的Directive。
* 有两种不同的Directive，structural和attribute：
  * structural会通过添加、删除、替换DOM元素来修改布局，这类directive一般带有`*`前缀。
  * attribute主要用来修改现有元素的样子和行为，与HTML自身的attribute类似。

### Services

* service就是普通的类，好的实践是将组件类中的逻辑部分提出到service实现重用。

### 依赖注入

* Angular根据组件类的构造函数参数来推测这个组件需要依赖哪些service。
* `injector`：类似一个service实例的容器，Angular创建一个组件类的实例时，根据构造函数参数来确定需要的service并向injector发出请求，如果injector内部已有该service的实例则之间返回，没有的而话则会创建一个放入容器并返回。
* injector如何知道怎么创建一个service？通过provider！provider是创建service的配方，会创建或直接返回一个service。而provider是在定义module或component在metadata里进行注册的：
  * 在module里注册，则这个module所有引用service的地方都是共享同一个实例。
  * 在component里注册，则每次实例化一个新组件时也会同时创建这个service的新实例。

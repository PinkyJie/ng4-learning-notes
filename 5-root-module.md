The Root Module
=======

* 每个Angular应用都必须有一个root module用来bootstrap，这个module的名字可以随便起，一般都叫`AppModule`。

### imports

* 只有被`@NgModule`decorate过的类才能放进imports属性的数组中。
* 只要应用需要在浏览器中运行，一般都需要在root module的imports中声明依赖`BrowserModule`，该module来自`@angular/platform-browser`。

### declarations

* 每个component必须声明在某个module的declarations中。
* 只有3种类型的类可以出现在declarations数组中：component、directive、pipe。

### bootstrap

* 整个应用的启动开始于对root module进行bootstrap，该数组中的组件会被创建并插入到DOM中去。
* bootstrap数组中可以放多个组件，但一般只放一个。

### bootstrap过程

* 一般在单独的文件中（如`main.ts`）对root module进行bootstrap。

``` typescript
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule }              from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

* bootstrap的流程是，通过传入的root module的metadata找到其bootstrap数组定义的组件，创建组件的实例并根据组件的metadata中定义的selector将其插入DOM中去。

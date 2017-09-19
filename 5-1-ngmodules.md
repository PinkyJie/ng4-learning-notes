NgModules
========

* `@ngModule`的metadata主要作用：
  * 声明哪些component、directive、pipe属于这个module，要使用本module内定义的C、D、P则必须将其放入`declarations`中去。
  * 指定哪些是public出去可以给别人用的，定义在`exports`中即可。
  * 从别的module导入component、directive、pipe给自己用。将依赖的module加入`imports`中，则这个module包含的所有public的CDP都对本module可见了。
  * 提供service给整个应用使用，要使用本module内定义的service则必须将其放入`providers`中去。只要在module层级的`providers`里声明，则service就整个应用可见，如果在component层级的`providers`声明，则它只对这个component可见。注意：在非layload的module中不存在module层级的service，因为各个module和root module使用同一个injector，但lazyload的module中有自己的injector。

### 两种Compile方式

* JIT(Just In Time)：
  * `platformBrowserDynamic().bootstrapModule(AppModule);`
  * 动态的，编译器在浏览器中编译整个应用并启动。
* AOT(Ahead Of Time)：
  * ``` typescript
      import { AppModuleNgFactory } from './app/app.module.ngfactory';
      platformBrowser().bootstrapModuleFactory(AppModuleNgFactory);
    ```
  * 静态的，编译器预编译生成很多工厂类，如`app.module.ngfactory`，直接bootstrap工厂类即可。
  * 由于整个应用都已经被预编译，最终生成的给浏览器使用的代码已经不含有编译器，所以体积会变小不少。
  * 由于浏览器拿到的代码已经是立即可执行的，省去了在浏览器中动态编译的时间，所以性能也会提升不少。
  * 注意：不管是JIT还是AOT都会编译生成`.ngfactory`的文件，区别是JIT在浏览器中动态生成，生成结果保存在内存里，而AOT是提前生成，直接生成物理文件。
  * module本身不需要关注自己是被JIT编译还是AOT编译的，控制编译的逻辑是放在加载module的`main.ts`中的。

### `BrowserModule`

* 所有基于浏览器的应用都必须`imports`这个module，除此之外它还提供基本的directive支持，如`ngIf/ngFor`。严格来讲，其实`ngIf/ngFor`等来自于`CommonModule`，而这个module是`BrowserModule`的一个依赖。

### feature module

* 为什么需要？如果有两个定义在不同地方的同名（类名相同）directive，它们作用于同一个attribute，则后定义的会覆盖先定义的那个的行为。想解决这种冲突，就需要定义不同的module将它们放入不同的module中。
* 将重名的directive放入单独的module中，只要不把它暴露在`exports`中，则这个directive对外部就是不可见的，这样就解决了冲突。

### Lazy-loading

* root module只需要`imports`初始时需要加载的module即可，动态加载的module不必引入。
* root module的appComponent中提供一个`<router-outlet>`用来显示子路由的component。
* 每个module（包括root）都需要定义一个routing module，里面定义路由信息，并调用`RouterModule`的静态方法将其暴露除去，如
``` typescript
  @NgModule({
    // root module 使用 .forRoot, feature module使用 .forChild
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule]
  })
```
* 每个module都必须在自己的`imports`中引入这个新定义的routing module。
* 每个feature module不需要再`exports`自己的component，因为这个信息已经在路由信息中定义好了。

### shared module

* 将公用的CDP定义在共享的module中，然后再统一`exports`出去。
* 一些公用的module，如`CommonModule/FormsModule`也可以引入到共享module中，然后再re-export出去，这样那些`imports`共享module的其他module也可以使用这些公用的CDP了。

### core module

* 虽然某些service在多处使用，但如果需要其在整个应用中是单例的话，则不能将它放入共享的module中，因为service的共享是基于依赖注入实现的，将其放入共享module多次引用就会整个应用中出现多个实例。解决方法是将其放入一个core module，并在root module中引入，保证只被引入一次。
* 使用`forRoot`方法可以在引入module的同时对包含的provider进行配置。在自定义的module中，定义`forRoot`令其返回一个`ModuleWithProvider`对象，包含module和provider信息。
``` typescript
  static forRoot(config: UserServiceConfig): ModuleWithProviders {
    return {
      ngModule: CoreModule,
      providers: [
        {provide: UserServiceConfig, useValue: config }
      ]
    };
  }
```
* 如何防止一个module被多次应用？在module的构造函数中检查即可。构造函数里的参数表明angular会尝试在父级的injector中查找这个module，如若在root module中引用，则没有父级injector，如若同时在root module和lazyload的module中引用，则父级的injector就是root module的injector，则构造函数就会报错。里面的`@Optional`代表找不到也不要报错，`@SkipSelf`表明跳过自身。
``` typescript
  constructor (@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error(
        'CoreModule is already loaded. Import it in the AppModule only');
    }
  }
```


NgModule FAQs
=============

### `declarations`

* 只有components、directives和pipes才可以放进去。
* 在别的module中已经declare过的不能重复declare。

### `imports`

* 只能import被`@NgModule`修饰过的module。
* 如果本module的component template中需要引用别的module中`exports`出来的东西，则需要将需要的module放入`imports`。
* 只能在root module中引入`BrowserModule`。
* 如果一个module被多次引用，angular只会引入一次，对于后面的引入直接使用缓存。但需要自己便面循环引用的问题。

### `exports`

* 除了components、directives和pipes可以被`exports`之外，module也可以被整个`exports`出去。`exports`一个module会把这个module中`exports`的东西re-exports出去。
* 如果不放进`exports`中，则该class只对module内可见。

### `providers`

* 在一个被bootstapped（非lazyload）的module中定义的service是整个应用全局可见的，这个时候只有一个全局的application injector。
* 当一个lazyload的module被加载时，会为这个module创建一个新的运行上下文，这个上下文有自己的module injector，这个injector是全局application injector的子类。
* 当两个不同的module存在同名的service时，后定义的那个会覆盖先前定义的。
* 当一个module自己含有service A，但同时又从另一个module里import了另外一个service A，自己的service A会取胜，因为providers的处理顺序把自己显式定义在自身`declarations`里的service放在最后处理。
* 正是由于这种处理的顺序性，推荐公用的service都在root module里引入，如需定制引入的servcie，也推荐直接在root module里进行定制。
* 只有在service仅对某component及其子component私有时，才需要将该service声明在component级别的`declarations`中。

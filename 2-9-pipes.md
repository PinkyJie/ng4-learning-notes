Pipes
=======

* 就是ng1里的filter，用于在模板里对数据进行再处理。
* pipe可以接收参数，通过`:`隔开，多个参数也是同样用`:`隔开，参数可以是字面字符串，也可以是来自组件的property变量。
* pipe可以chain。

### 自己写pipe

* 一个被decorator `@Pipe`修饰的类，metadata里必须带有`name`，这个属性就是在模板中使用该pipe时的名字。
* 必须实现接口`PipeTransform`的`transform`方法，这个方法的参数就是原始值以及pipe的各个参数。
* 与directive类似，自定义的pipe必须由module的`declarations`暴露出去才能使用。

# pure与impure pipe

* pipe默认是pure的，也就是说只会在值或参数发生变化时才会重新执行（不同于普通的change detection，并不是所有的异步事件都会导致pipe逻辑被重新执行，为了性能），这就导致，如果原始值是一个数组，那么王数组中添加或删除元素并不会触发pipe重新执行，因为数组的引用没有变化。如果想要pipe重新执行，必须在数组内元素变化时重新给数组变量赋值。
* 可以通过metadata中的`pure: false`参数来定义一个impure的pipe。而impure的change detection与正常的一样，即所有异步事件发生后pipe的逻辑都会被重新执行。显示impure的pipe会危害性能。
* 内置的impure pipe例子：`AsyncPipe`，它的原始值是Promise或Obserable，这个pipe会自动订阅这个源，在消息到达时显示结果，并在销毁时自动取消订阅。

### 不再提供`filter`和`orderBy`的pipe

* 性能考虑，这两个都是impure，执行非常频繁。
* 推荐将此类功能移到组件中实现。

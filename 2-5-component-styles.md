Component Styles
=========

* angular允许定义组件级别的样式，在组件的metadata中`styles`接受一个字符串数组，任何合法的CSS都可以放在里面，只对该组件有效。

### 组件内使用特殊选择器定义其container的样式

* container指的是组件的selector选中的父模板中的元素。
* 使用`:host`可以选中该组件的container，加上括号`:host(.xxx)`表示两个选择器叠加在一起。
* 使用`:host-context(.xxx)`，沿该组件一直向上查找具有某个特定class的元素，知道document root为止。通常用于组件主题的定制，如body上带有特定的主题class时，定制该组件在这个主题下的样式。
* 使用`/deep/`或`>>>`使得定义的样式不只使用于当前模板，还可以应用到所有的子组件上，包括view child和content child。

### 定义组件样式的多种方法

* metadata中的`styles`数组，通常只放一个字符串。
* metadata中的`styleUrls`数组，可以加载多个CSS文件路径。注意这里的相对路径是相对于根HTML所在路径的，而不是当前文件的路径。若要使用当前的文件的相对路径，在路径前加上`./`即可。
* 使用webpack的css-loader时，可以直接`styles: [require('my.component.css')]`，因为webpack会负责将其转化为CSS字符串。
* 组件的模板定义中可以直接写`<style>`和`<link>`。

### angular如何实现scoped CSS

* metadata中有个`encapsulation`属性，它有几个取值：
  * `ViewEncapsulation.Native`：使用浏览器内置的Shadow DOM来实现。
  * `ViewEncapsulation.Emulated`：默认选项，angular通过给元素追加自定义的attribute来实现，如：
    * `<h3 _nghost-xxx>`，追加在组件的container元素上。
    * `<p _ngcontent-xxx>`，追加在组件的view元素上，标识这个元素属于哪个container。
  * `ViewEncapsulation.None`：直接使用全局CSS样式。






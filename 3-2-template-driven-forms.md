Forms
=========

* Typescript使用`?`跟在入参变量名后，表明这是一个可选的参数。
* HTML元素的显示隐藏：ng1的`ng-hide/ng-show`已没有，常用的方法是直接绑定HTML的原生hidden属性，如`<div [hidden]="..."`。

### 使用`ngModel`实现双向绑定

* 基本语法：`[(ngModel)]="..."`。
* 要想在模板里访问form上的元素或做输入校验，需要使用template reference variable：`<form #heroForm="ngForm">`。这么写的原因是一般引入了`FormsModule`后，angular就会自动将`NgForm`这个directive绑定到HTML原生的form元素上，而通过将`ngForm`赋值给变量`#heroForm`，这个变量就不再指向原生的form，而是指向`NgForm`这个directive改造后的form（实际上就是`NgForm`这个directive的实例本身）。至于`ngForm`这个名字是由directive的`exportAs`来决定的。
* 使用ngModel必须在该表单元素上添加`name`属性。注意`<label>`的for使用的是`id`属性。

### 使用`ng-*`来定义表单校验样式

* angular会识别表单元素上的校验属性，如`required`，并将相关的校验方法添加到相应的model上。
* `ng-touched/ng-untouched`：点一下input框然后点别的地方，此时untouched变为touched。
* `ng-pristine/ng-dirty`：input框内容是否被修改。
  * 这个“改变”针对的是component的绑定数据显示后是否修改过。比如通过代码逻辑修改model后页面input框随之更改，但这不会触发dirty。
  * 但是若代码逻辑修改model前就已经dirty，修改model并不会将其状态恢复为pristine。如需实现这个功能，需要调用`heroForm.reset()`方法。
* `ng-valid/ng-invalid`：校验是否通过。

### 模板上获取表单元素的状态

* 在input元素上使用template reference variable使用，如`<input #name="ngModel">`，这里的`ngModel`与上面的`ngForm`类似，也是通过directive改变变量的指向，这里的`ngModel`为directive `ngModel`定义的`exportAs`。
* 使用变量来获取校验相关的状态，如`name.valid`或`name.pristine`。
* 表单元素的校验错误信息：`name.errors.required`或`name.errors.minlength/maxlength`，名称都与DOM自带的校验属性名称一致，值都是布尔的。
* 使用表单的template reference variable来获取整个表单的校验状态：`heroForm.form.valid`。

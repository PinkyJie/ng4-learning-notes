User Input
======

* Angular的事件绑定语法：`<button (click)="onClickMe()">`。
* 使用`$event`变量将事件对象传入绑定的handler中：`<input (keyup)="onKey($event)">`。
* 直接传入`$event`并不是一个好的实践，会导致模板和组件强关联，更好的方法是使用template reference variable：`<input #box (keyup)="0">`。
  * 使用`#`语法定义变量，则整个模板就可以直接使用`box`来访问该元素。
  * 必须要添加`(keyup)`绑定，因为Angular只有在异步事件发生后才会去更新绑定的值。
* key filter：`<input #box (keyup.enter)="onEnter(box.value)">`，可以直接对输入键值进行过滤，只保留回车键。
* handler里可以写多个语句，用分号隔开即可。

Reactive Forms
=========

### 与Template-driven Forms的区别

* Template-driven Forms是异步的。因为所有的表单元素都是directive根据模板创建的，为避免“changed after checked”报错，必须等下一个tick才能修改表单元素。
* Reactive Forms是同步的。因为所有的表单元素都是在类代码里创建好的，可以随时访问。

### 常用的几个类

* `AbstractControl`：底下三个的抽象父类。
* `FormControl`
  * 对应表单控件，这个directive用于创建和管理一个表单元素，如`<input>`。
  * 组件类中使用`name = new FormControl()`定义一个表单元素。
  * 模板中使用`<input [formControl]="name">`将`<input>`通过**字符串`name`**进行绑定，这里的`name`对应的就是组件类中的`FormControl`实例的变量名。
  * 常用属性：`value/status/pristine/dirty/untouched/touched`。
  * 通过订阅其属性`valueChanges`（一个RxJS的Observable）可以监测值的变化。
* `FormGroup`
  * 如果存在多个`FormControl`，通常使用`FormGroup`包裹住它们方便管理。包含多个表单元素的`<form>`就对应一个`FormGroup`。
  * 组件类中使用`heroForm = new FormGroup ({ name: new FormControl() });`来定义一个表单组。
  * 模板中使用`<form [formGroup]="heroForm">`来绑定。
  * 注意：如果在`FormGroup`底下使用`FormControl`，模板中必须使用`formControlName="name"`来进行绑定。与`[formControl]="name"`的区别是前者告诉angular在group中寻找特定的控件，而后者是可以独立工作的。
  * 常用的几个属性和方法：
    * `.get(name)`：获取该group下的某个FormControl实例，参数name支持`xxx.xxx`（嵌套group）。注意这个方法返回的类型是`AbstractControl`，如果需要具体类型，需要强制转换。
    * `value`：返回一个object，包含该group下所有的表单元素的值。
    * `status`：整个表单的校验状态。
* `FormArray`
  * 包含任意多（可以是零）个`FormGroup`。比如，一个用户可能有多个地址，一个居住地址，一个账单地址。每个地址都是一个group（因为包含多个字段），而整个地址字段就是一个array。
  * 模板里使用`formArrayName="xxxx"`进行绑定。
  * 在使用`ngFor`进行循环时，需要使用`<div *ngFor="let address of secretLairs.controls; let i=index" [formGroupName]="i">`。注意：
    * 循环的主体不是`FormArray`本身，而是它的`.controls`属性。
    * 每个循环体必须指定`formGroupName=""`来完成绑定，直接采用循环索引值。
    * 循环的每个地址部分仍然需要使用`formControlName=""`来绑定，与group时的用法一致。

### 使用`FormBuilder`简化创建流程

* 当需要同时创建大的表单时，使用`FormBuilder`可以让代码更简洁。
* 组件类中的代码如下。使用`FormBuilder`的`group`方法可以快速定义一堆表单元素，而不用手动写`new FormControl`。
  ``` typescript
    export class HeroDetailComponent3 {
      heroForm: FormGroup; // <--- heroForm is of type FormGroup

      constructor(private fb: FormBuilder) { // <--- inject FormBuilder
        this.createForm();
      }

      createForm() {
        this.heroForm = this.fb.group({
          name: '', // <--- the FormControl called "name"
        });
      }
    }
  ```
* 上面例子中直接使用表单控件的初始值（空字符串）来定义表单元素，除此之外，也可以使用一个数组`name: ['', Validators.required ]`，第二个参数定义校验规则，检验规则自身也可以是一个包含多个规则的数组。
* group支持嵌套，如可以在上面的group中加入`address: this.fb.group({ street: '', city: '', state: '', zip: '' }),`，同时，模板中也必须反映同样的结构，即`[formGroup]`的结构中嵌套另一个`[formGroup]`。
* 使用`fb.array([])`来创建一个`FormArray`，注意里面数组的每个元素是`FormGroup`的实例。与`setValue`类似，如果要给更新`FormArray`的值，需要使用`setControl`。

### 将数据应用于表单

* data model和form model：data model通常是来自于API的数据，而form model对应的是HTML中显示的表单的数据。从API拿到数据后，需要将data model应用于form model，而用于对于表单的更改是直接作用于form model的，不会对data model产生影响。
* 使用`FormGroup`的`setValue/patchValue`来将data model应用于form model，两个方法都接收一个对象，区别是前者会检查是否有缺失的字段并报错，而后者不会。
* 通常data model每次发生变化时就需要使用`setValue/patchValue`来更新form model，同时还需要重置表单元素的校验状态，直接调用`FormGroup`的`reset()`即可，该方法可以同时更新值和重置状态。
* 在save的时候同样需要把form model的数据更新会data model，从form model读取数据构造data model要注意，一些情况下需要clone出新对象，防止两者公用一份copy导致用户在页面上的后续更改不小心改动data model。

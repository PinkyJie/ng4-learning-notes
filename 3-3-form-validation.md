Form Validation
=======

### 表单校验

* 在模板里写死：
  * 使用ngModel的reference variable提供的`variableName.errors.xxx`配合`ng-if`在模板中显示不同的校验信息。
* 在代码里动态处理：
  * 使用`@ViewChld`在组件中获取form实例，如`@ViewChild('heroForm') currentForm: NgForm;`，传入的是reference variable的变量名，而form实例的类型就是`NgForm`。
  * form的实例会发生变化（？），所以需要周期性的重新获取form实例。常用的方法是在组件的声明周期`ngAfterViewChecked()`（子组件NgForm的property发生变化了）中重新将`currentForm`变量（已是最新）赋值给自定义的其他变量。
  * 通过订阅`ngForm`的property `valueChanges`来获取最新的表单元素值变化。也可以通过`NgForm`的`.form.get("name")`来获取name对应的元素的control model（即NgModel实例）。
  * 下面的例子中将预设的校验报错信息放在`validationMessages`变量中，将当前需要页面显示的报错信息放在`formErrors`变量中，模板只需要关注`formErrors`，大大简化。
  ```typescript
    heroForm: NgForm;
    @ViewChild('heroForm') currentForm: NgForm;

    ngAfterViewChecked() {
      this.formChanged();
    }

    formChanged() {
      if (this.currentForm === this.heroForm) { return; }
      this.heroForm = this.currentForm;
      if (this.heroForm) {
        this.heroForm.valueChanges
          .subscribe(data => this.onValueChanged(data));
      }
    }

    onValueChanged(data?: any) {
      if (!this.heroForm) { return; }
      const form = this.heroForm.form;

      for (const field in this.formErrors) {
        // clear previous error message (if any)
        this.formErrors[field] = '';
        const control = form.get(field);

        if (control && control.dirty && !control.valid) {
          const messages = this.validationMessages[field];
          for (const key in control.errors) {
            this.formErrors[field] += messages[key] + ' ';
          }
        }
      }
    }
  ```

### Reactive Form

* angular里的两种form：Template Driven Form - `FormsModule`，Reactive Form - `ReactiveFormsModule`。
* 区别：
  * Template Driven Form，已模板为主，模板中定义好各个表单元素，校验规则，然后angular在运行时解析模板，绑定`NgForm`以及相应的control model。
  * Reactive Form，以代码为主，代码中定义好表单的各个control model，然后angular运行时将代码中的model与模板进行绑定。

### Reactive Form的构建

* 模板：
  * 表单定义：`<form [formGroup]="heroForm">`，这里的`heroForm`为组件代码中定义好的control model。
  * 表单元素定义：`<input type="text" id="name" class="form-control" formControlName="name" required >`：
    * `name`改为`formControlName`，与组件代码中定义好的model进行绑定。
    * 除`required`其他校验规则统统挪到了代码中进行定义。其实`required`代码里也定义了，放这里是为了样式考量。
    * 不再使用`[(ngModel)]`进行双向绑定。
* 组件类：
  * 使用`FormBuilder`这个service来定义表单。
  ```typescript
    heroForm: FormGroup;
    constructor(private fb: FormBuilder) { }

    ngOnInit(): void {
      this.buildForm();
    }

    buildForm(): void {
      this.heroForm = this.fb.group({
        'name': [this.hero.name, [
            Validators.required,
            Validators.minLength(4),
            Validators.maxLength(24),
            forbiddenNameValidator(/bob/i)
          ]
        ],
        'alterEgo': [this.hero.alterEgo],
        'power':    [this.hero.power, Validators.required]
      });

      this.heroForm.valueChanges
        .subscribe(data => this.onValueChanged(data));

      this.onValueChanged(); // (re)set validation messages now
    }
  ```
  * 使用`FormBuilder`构建的表单实例不在属于`NgForm`类型，而是`FormGroup`类型。
  * 由于没有了`[(ngModel)]`，那么每次hero数据发生变化时都需要调用`buildForm()`，如最开始的`ngOnInit()`以及后续新建hero时：
  ``` typescript
    addHero() {
      this.hero = new Hero(42, '', '');
      this.buildForm();
    }
  ```

### 自定义校验

* 如果应用于Reactive Form，则只需要定义一个函数即可。这个函数需要返回另一个函数：入参是表单的control（用于获取表单元素的值），返回值是一个ValidationError的对象或null。
* 如果应用与Template Driven Form，则需要定义一个directive：
  * directive的metadata中providers需声明`[{provide: NG_VALIDATORS, useExisting: ForbiddenValidatorDirective, multi: true}]`，其中使用`useExisting`的目的是让其直接使用模板里传入的参数。
  * 需实现接口`Validator`和`ngOnChanges`，前者里的`validate()`函数就是校验逻辑，与上面Reactive Form时定义的函数的返回函数格式一致。后者`ngOnChanges()`用来响应`@Input`值变化进而更新校验逻辑。

Displaying Data
=======

* 展示数据最简单的方式就是：将HTML页面元素与组件的属性就行绑定。
* Typescirpt定义类的property，以下两种方式等价：

``` typescript
// variable assignment style
export class AppComponent {
  title = 'Tour of Heroes';
  myHero = 'Windstorm';
}

// declare and initialize the properties using a constructor
export class AppCtorComponent {
  title: string;
  myHero: string;

  constructor() {
    this.title = 'Tour of Heroes';
    this.myHero = 'Windstorm';
  }
}
```

* `*ngFor`：注意`*`前缀，作为属性放在需要复制的HTML元素上。
* Typescript定义类的property简写：
``` typescript
export class Hero {
  constructor(
    public id: number,
    public name: string) { }
}

// equals to
export class Hero {
  public id: number;
  public name: string;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }
}
```

* `*ngIf`：注意前缀，直接添加或移除整个DOM元素，并不是隐藏。若元素内含有多个绑定，则对性能是有好处的。

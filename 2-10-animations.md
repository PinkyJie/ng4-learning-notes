Animations
========

* angular的动画使用标准的Web Animations API，一些浏览器需要polyfill。
* 动画的module：`BrowserAnimationsModule`，其他一些需要引入的方法`trigger/state/style/transition/animate`。

### 一个例子

* 一个定义在metadata的动画：
``` typescript
  animations: [
    trigger('heroState', [
      state('inactive', style({
        backgroundColor: '#eee',
        transform: 'scale(1)'
      })),
      state('active',   style({
        backgroundColor: '#cfd8dc',
        transform: 'scale(1.1)'
      })),
      transition('inactive => active', animate('100ms ease-in')),
      transition('active => inactive', animate('100ms ease-out'))
    ])
  ]
```
* 基本理念：将动画的不同状态定义成不同的`state`，每个状态有自己的样式，然后通过`transition`定义不同状态如何切换，通过`animate`定义切换时的动画效果。最后通过`trigger`将这一系列定义聚合起来，起个名字。这样，就可以使用绑定`[@triggerName]`在模板中触发动画了，由绑定的右侧来确定需要切换到哪个状态。
* transition定义中，除了`=>`还支持`<=>`。
* transition的第二个参数可以是个数组，手动定义样式，如下。但当动画结束后这两个样式都不会保留，因为它没有在state中定义，state中定义的才是动画结束的最终状态。
``` typescript
  transition('inactive => active', [
    style({
      backgroundColor: '#cfd8dc',
      transform: 'scale(1.3)'
    }),
    animate('80ms ease-in', style({
      backgroundColor: '#eee',
      transform: 'scale(1)'
    }))
  ])
```
* transition中的state可以使用通配符：
  * `*`匹配所有状态。
  * `void`匹配元素未被显示或已经移除时的状态，多用于进入或移除时的动画。如`Enter: void => *`，`Leave: * => void`，也可以直接使用别称：`:enter`或`:leave`。
* `animate`函数指定动画时间的方式与CSS类似：`animate('duration delay ease-func')`。

### keyframes动画

* 使用`keyframes`函数定义，参数为一个`style`的数组，整个keyframes的结果作为`animate`的第二个参数。
* 每个style都需要定义一个`offset`，一个0到1的数字，相当于原生keyframes的百分比，若不定义则默认均分。

### 同时开始多组动画

* 场景：同时animate多个CSS属性，各个属性的timing或ease不一样。这时可以使用`group`函数：
``` typescript
  group([
    animate('0.3s 0.1s ease', style({
      transform: 'translateX(0)',
      width: 120
    })),
    animate('0.3s ease', style({
      opacity: 1
    }))
  ])
```

### 动画的回调

* 使用绑定`(@triggerName.start)="handler($event)"`或`(@triggerName.done)="handler($event)"`。

https://javascript.info/event-delegation

利用事件冒泡和事件捕捉，可以实现更优秀的事件处理模式：事件委托

如果有很多元素需要同样的事件处理，与其分配给每个元素handler，不如给它的祖先元素一个统一单个handler。当点击子元素，事件会冒泡到它的祖先元素，然后利用event.target来判断如何处理。

父元素捕获后代元素的事件，后代元素将它们的事件冒泡到父元素

典型的例子设置event到`<ul>`, 而不是`<li>`. (但是现在都用模板，数据绑定，不会复制粘贴`<li>`来分配handler了吧)

优点:
1. 节约内存, 不用重复创建多个event handler
2. 不用copy & paste 绑定 event handler, 当频繁添加删除子元素的时候，不需要再操作handlers
3. 对于动态生成的元素，不用再rebind handler
4. 如果子元素之间的handler不相同，更应该把整体的handler逻辑写在父元素，避免出错

# 原生JS 实现事件委托

```js
document.getElementById("parent-list").addEventListener("click", function(e) {
  // event.target返回了事件发生的目标元素, 利用event.target来判断如何处理
  if(e.target && e.target.nodeName == "LI") {
    // List item found!  Output the ID!
    console.log("List item ", e.target.id.replace("post-", ""), " was clicked!");
  }

  // 可以使用Element.matches, querySelector来筛选元素
  if (e.target && e.target.matches("li.classA")) {
    console.log("Anchor element clicked!");
  }
});

<ul id="parent-list">
	<li id="post-1" class="classA">Item 1</li>
	<li id="post-2">Item 2</li>
	<li id="post-3">Item 3</li>
</ul>
```

# jQuery `.on()`

```js
<div class="container">
  <button class="button">Click Me</button>
</div>
<button class="add-button">Add a new button</button>

$('.add-button').click(function() {
	$('.container').append('<button class="button">Click Me</button>');
})

// 对于新创建的元素，也能有click事件，因为是委托在祖先元素，而且single handler处理全部元素
$('.container').on('click', '.button', function() {
	alert('Hello!');
})

// 这种方式对于每个selector匹配上的元素都会创建一个单独的handler, 问题:
// 1. 许多元素就会创建相同的handlers, 浪费内存
// 2. 动态创建的元素不会有handler, 除非rebind the handler
$("button.button").click(function() {
    alert(1);
});
```

`.on( events [, selector ] [, data ], handler)`
> **events**: event type + namespaces(optional) "click.myPlugin".
**selector**: A selector string to filter the descendants of the selected elements that trigger the event. If the selector is null or omitted, the event is always triggered when it reaches the selected element.
后代元素选择器，专门方便事件委托的？
如果不设置，事件到底选中元素时总被触发, 即所有后端元素都触发？
[jQuery .on() API](http://api.jquery.com/on/)

[Difference between .on('click') vs .click()](https://stackoverflow.com/questions/9122078/difference-between-onclick-vs-click)

`.on()`还有一个优势，是可以用namespace:
`$("#element").on("click.someNamespace", function() { console.log("anonymous!"); });`

如果对于handler function有reference，可以用off() unbind:
`$("#element").off("click", handler)`

但是如果没有，就需要用namesapce
`$("#element").off("click.someNamespace");`

https://jsfiddle.net/tinachen/um0ngd41/10/
为啥点击LI的时候只有LI的event, 没有冒泡到UL?



---

# React

[Event delegation with React](https://medium.com/shehacks/react-class-instantiation-and-stateless-components-a5d0e10a26b9)

React不是在创建元素后，在给DOM添加event listener, 而是在元素render的时候就提供了listener

和jQuery中的事件委托没啥区别。如果需要频繁创建&删除子组件，添加&移除event listener就可以用事件委托。修改的时候也直接修改父元素中的listener就行

---

[Why React discourage event delegation?](https://blog.cloudboost.io/why-react-discourage-event-delegation-2b5fe3f52bea)

> Never append **custom fields** into React’s events. Keep using props to pass data. When the app get complex, you should consider using Redux, not using event delegation in JQuery style
使用props来传递事件，或者Redux，不要使用jQuery风格的事件委托

React does support event bubbling and capturing with its synthetic events.
React中的synthetic events不支持事件冒泡和事件捕获

React discourage event delegation with custom field in event(?).
What React intentionally discouraged is to put custom data in event.
React阻止在事件中放入custom data

* 什么是custom data?

如果经常使用事件冒泡，会需要保存一些目标元素的属性，这就是custom data。
eg: 事件委托在Container中，如果需要显示子元素的title, 就需要把title 加到event custom data中

```js
class Card extends React.Component {
  onClickHandler = (e) => {
    e.title = this.props.title;
  }
  render() {
    const { icon, title, excerpt } = this.props;
  	return (
      <div
        className='card'
        onClick={ this.onClickHandler }>
        <img className='icon' src={ icon }/>
        <h2 className='title'>{ title }</h2>
      </div>
    );
  }
}

class App extends React.Component {
  // Card的click event委托给App处理
  onClickHandler = (e) => {
    alert(`Card clicked: ${e.title}`);
  }
  render() {
  	return (
      <div onClick={ this.onClickHandler }>
        <Card
          icon='https://facebook.github.io/react/img/logo.svg'
          title='Why React discourage event delegation?'/>
      </div>
    );
  }
}

```

* Custom event delegation 的缺点

违反了React的原则：

1. New data flow is created

**One way data flow** 是React的核心思想之一。但是在自定义事件委托中，数据通过events传递给了父组件。事件冒泡就成为了另一条数据流。
当应用变大时，很难维护两条数据流，而且event中的data不能被React开发者工具捕获，使用事件冒泡很难debug

2. Implicit dependency is introduced 隐式依赖被引入

React试图让所有依赖都是显式的(explicit), 即使这意味着多写点代码。但是自定义事件委托在父元素和子元素之间引入了隐式的依赖。

比如在我们的例子中，App虽然没有传递函数给Card, 但是需要了解Card的行为，需要知道：

1. There is a custom field `title` in click event 需要知道捕获的Card click event中是有一个`title`属性的
2. The clicked card title need to be alert 知道这个card title需要被弹出

如果不使用自定义事件委托，可以使用：
1. 从App传递click handler
2. 在Card中定义click handler. 因为Card这个组件在创建很多实例的时候，只要把handler写在构造函数中(好像是吧)，也共用同一个handler instance, 也就不存在重复creat & bind浪费内存的问题了。所以事件委托在这些新框架上没有优势嘛!

这两种方法都显式声明了依赖，代码可读性更高。

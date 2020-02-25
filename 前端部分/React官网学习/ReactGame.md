# 遇到的问题：

1. 【每一个 React 元素都是一个 JavaScript 对象，程序中可以把它当保存在变量中或者作为参数传递。】**？？？**



# 前置准备：

### 1、 环境准备

1. 使用脚手架创建项目

```shell
# 使用react脚手架创建项目【npx使用后不会留下任何残留】
npx create-react-app my-app
```

2. 删除src下的所有文件

```shell
cd my-app
cd src

# 如果你使用 Mac 或 Linux:
rm -f *

# 如果你使用 Windows:
del *

# 然后回到项目文件夹
cd ..
```

3. 在`src`下创建`index.css`和`index.js`，并放入下方代码

&emsp;&emsp;`js`文件

```js
class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {/* TODO */}
      </button>
    );
  }
}

class Board extends React.Component {
  renderSquare(i) {
    return <Square />;
  }

  render() {
    const status = 'Next player: X';

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}

class Game extends React.Component {
  render() {
    return (
      <div className="game">
        <div className="game-board">
          <Board />
        </div>
        <div className="game-info">
          <div>{/* status */}</div>
          <ol>{/* TODO */}</ol>
        </div>
      </div>
    );
  }
}

// ========================================

ReactDOM.render(
  <Game />,
  document.getElementById('root')
);
```

&emsp;&emsp;`css`文件

```css
body {
  font: 14px "Century Gothic", Futura, sans-serif;
  margin: 20px;
}

ol, ul {
  padding-left: 30px;
}

.board-row:after {
  clear: both;
  content: "";
  display: table;
}

.status {
  margin-bottom: 10px;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.square:focus {
  outline: none;
}

.kbd-navigation .square:focus {
  background: #ddd;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

4. 拷贝下方代码到`index.js`的顶部

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
```



# 一、Rract概述



### 1.1 React简介

&emsp;&emsp;React 是一个**声明式，高效且灵活**的用于构建用户界面的 JavaScript 库。使用 `React`可以将一些**简短、独立的代码片段组合成复杂的 UI 界面**，这些代码片段被称作`“组件”`。



### 1.2 React的一些子类



#### 1.2.1 React.Component子类

```js
class ShoppingList extends React.Component {
  render() {
    return (
      <!-- 组建中可以嵌套组件，达到组合的效果 -->
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}
// 用法示例: <ShoppingList name="Mark" />
```

**说明：**

- `ShoppingList`：组建类
- `render()`:返回需要展示的内容【此处使用的是`JSX`(一种对渲染内容的轻量级描述)】

- `props`：即通过标签属性传入的参数。
- `{this.props.name}`：`JSX`中可以任意的使用js表达式，需要用`{}`括起来。【每一个 React 元素都是一个 JavaScript 对象，程序中可以把它当保存在变量中或者作为参数传递。】**？？？**

> &emsp;&emsp;`JSX`会将`<div /> `会被编译成 `React.createElement('div')`
>
> 上述语法等同于：
>
> ```js
> return React.createElement('div', {className: 'shopping-list'},
>   React.createElement('h1', /* ... h1 children ... */),
>   React.createElement('ul', /* ... ul children ... */)
> );
> ```



### 1.3 构造函数和State

```js
class Square extends React.Component {
    //构造函数：初始化state
  	constructor(props) {
        super(props);
        this.state = {
            value: null
        }
    }

    render() {
        return (
            <button className="square" onClick={ ()=>alert(this.props.value) }>
                {this.props.value}
            </button>
        );
    }
}
```

- `state`:可以视为一个组件的私有属性

> **注意：**
>
> &emsp;&emsp;在 [JavaScript class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) 中，每次你定义其子类的构造函数时，都需要调用 `super` 方法。因此，在所有含有构造函数的的 React 组件中，构造函数必须以 `super(props)` 开头。



### 1.4 React多组件通信

&emsp;&emsp;当需要获取多个子组件数据，或者两个组件之间需要相互通信时，需要把子组件的`state`放到共同的父组件当中保存。

```js
class Board extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null),
    };
  }

  renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}//注意我们的方法也可以通过此种方式传递到子组件
      />
    );
  }
}
```

> 此时我们的子组件只需要`this.props.onClick`就能调用到这个方法了

### 1.5 函数组件

&emsp;&emsp;如果组件类中只有一个`render`方法，并且不包含`state`，这是可以使用函数组件。

```js
//改造后
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

> 注意：this.props都变成了props,并且`onClick`的箭头函数一起变成了更短的表达式。



# 补：

### 1.关于箭头函数中this

&emsp;&emsp;箭头函数中使用`this`调用外部的内容。
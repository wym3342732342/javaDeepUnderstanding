# 一、JSX简介

```js
const element = <h1>Hello, world!</h1>;
```

&emsp;&emsp;这种标签语法既不是字符串也不是`HTML`，其被称为`JSX`，是`js`语法的扩展。`React`推荐使用`JSX`。**`JSX`中可以使用任意的标签，并且可像Html一样嵌套。**



### 1.1 JSX中嵌入表达式

&emsp;&emsp;简单的说，`JSX`中可以使用`{}`来使用**变量**

```js
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

> 在`JSX`语法中，可以在`{}`内放置任何有效的`js表达式`



### 1.2 在if-else中使用JSX

&emsp;&emsp;在编译之后，`JSX`表达式会被转为普通的`js`函数调用，并且可以作为`js对象`返回

```js
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

> 利用这点就可以在不同的条件下显示不同的效果



### 1.3 JSX 特定属性

&emsp;&emsp;可以使用引号`""`指定字面量，也可以使用`{}`插入`js表达式`【不要再大括号外面加上引号】

```js
// 使用引号
const element = <div tabIndex="0"></div>;
// 使用大括号
const element = <img src={user.avatarUrl}></img>;
```

> **注意:**`JSX`语法更接近`js`而不是`HTML`，所以在`React DOM`中使用驼峰命名法来定义属性名称。



### 1.4 JSX防止注入攻击

&emsp;&emsp;使用`jsx`时，不必担心注入攻击，他会将所有待渲染内容先进行转译。可以保证所有内容在渲染之前都是字符串。



### 1.5 JSX表示对象

&emsp;&emsp;`Babel`会把`JSX`转译，其于使用`React.createElement()`函数创建对象完全等效。

```js
//使用jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);

// 使用React.createElement函数
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

&emsp;&emsp;`React.createElement`会预先执行一些检查，以防止编写出错代码。**实际上其创建了一个这样的对象：**

```js
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```





# 二、元素渲染

&emsp;&emsp;`React`元素是创建开销极小的普通对象。`React DOM`会负责更新`DOM`来与`React`元素保持一致。其于组件不同，组建由元素构成。



### 2.1 将元素渲染为DOM

&emsp;&emsp;`HTML`文件出需要有一个“根”节点。该节点的所有内容都有`React DOM`管理。通常只有一个根结点，当需要将React应用集成进已有应用时，就会出现任意多根结点的情况。

&emsp;&emsp;**将`React`元素渲染到根DOM节点中：**`ReactDOM.render()`

```js
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```



### 2.2 更新已渲染的元素

&emsp;&emsp;`React`元素是不可变对象，一旦被创建就无法更改它的子元素或属性。**就目前掌握的知识而言，只能使用计时器来达到更新的效果【多次调用`render`】：**

```js
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

> `React DOM`会将元素与他们之前的状态进行比较，**其只会进行必要的更新来使DOM达到预期的状态**。虽然例子中每一秒都会新建一个描述整个 UI 树的元素。



# 三、组件&Props



### 3.1 函数组件与class组件



**函数组件**：定义组件最简单的方式

```js
function Welcome(props) {//props代表属性对象，用于通信
  return <h1>Hello, {props.name}</h1>;
}
```



**类组件**：

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```



### 3.2 渲染组件

&emsp;&emsp;渲染组件，和我们渲染`JSX`的方式是一样的。

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

> 此时发生了什么：
>
> 1. 调用`ReactDOM.render`函数，传入`<Welcome name="Sara" />`作为参数。
> 2. React调用`Welcome`组件，并将`{name:'Sara'}`作为props传入。
> 3. `Welcome`组件将`<h1>Hello, Sara</h1>`元素作为返回值。
> 4. `React DOM`将`DOM`高效的更新为`<h1>Hello, Sara</h1>`。
>
> **注意：**组件名称必须以大写字母开头，React会讲小写字母开头的组件视为原声DOM标签。



### 3.3 组合组件

&emsp;&emsp;可以在组件中引用其他组件。给`ReactDOM.render`传参时也可以传入`JSX`。

```js
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}
function App() {
    return (
        <div>
            <Welcome name="Sara" />
            <Welcome name="Cahal" />
            <Welcome name="Edite" />
        </div>
    );
}
ReactDOM.render(
    <App />,
    document.getElementById('root')
);
```



### 3.4 提取组件

&emsp;&emsp;当一个组件中功能过多，这是就需要考虑将其变成`N多`轻量的小组件。这样方便维护和复用。



### 3.5 Props

&emsp;&emsp;`Props`及传入的属性对象，它是只读的，如果尝试改变它会报错。



# 四、State&生命周期



**演示类组件，时钟组件：**

```js
class Clock extends React.Component{
    render() {
        return (
            <div>
                <h1>你好, 世界!</h1>
                <h2>现在是北京时间 {new Date().toLocaleTimeString()}</h2>
            </div>
        );
    }
}
```

> &emsp;&emsp;每次组件更新时，`render`方法都会被调用，但是在**相同的DOM节点中渲染同一组件**，就会只有一个`class`实例被创建。这是我们就需要`state`或生命周期方法等特性.



**改造时钟组件：**

```js
class Clock extends React.Component {
    constructor(props) {//通过构造给state赋初值
        super(props);//必须将props传入父类
        this.state = {
            date: new Date()
        };
    }
    render() {
        return (
            <div>
                <h1>你好, 世界!</h1>
                <h2>现在是北京时间 {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        );
    }
}
```



### 4.1 组件类生命周期方法



**给组件添加计时器：**

```js
class Clock extends React.Component {
    constructor(props) {//通过构造给state赋初值
        super(props);//必须将props传入父类
        this.state = {
            date: new Date()
        };
    }
    componentDidMount() {
        //会在组件已经被渲染到DOM中后运行
        this.timerId = setInterval(//将计时器ID保存在timerId
            () => this.tick(), 1000
        );
    }
    componentWillUnmount() {
        clearInterval(this.timerId);//清除计时器
    }

    tick(){//重新为state赋值
        this.setState(
            {date:new Date()}
        )
    }

    render() {
        return (
            <div>
                <h1>你好, 世界!</h1>
                <h2>现在是北京时间 {this.state.date.toLocaleTimeString()}</h2>
            </div>
        );
    }
}
```

> 运行过程：
>
> 1. 将我们组件传给`ReactDOM.render()`，此时会初始化我们的组件类
> 2. 执行组件类构造函数
> 3. 调用`render`方法，更新DOM
> 4. 调用挂载方法
> 5. 当`state`改变时，会重新调用`render`方法。
> 6. 组件被移除，调用卸载方法



#### 4.1.1 挂载

&emsp;&emsp;`componentDidMount()` 方法会在组件已经被渲染到 DOM 中后运行

#### 4.1.2 卸载

&emsp;&emsp;当组件被移除时，`componentWillUnmount()`方法会被移除。



### 4.2 State

**使用`setState`必须要知道三件事：**

- **不要直接修改State中的属性**，构造函数是唯一可以给state赋值的地方。【直接赋值不会重新渲染组件】
- **state的更新可能是异步的**【React可能把多个`setState()`合并成一个再去调用】
- **state的更新会被合并**



#### 4.2.1 setState接收函数

&emsp;&emsp;因为React会把多个`setState`合并成一个，所以我们不能依赖`state`和`props`来更新下一个状态。这时我们就可以让`setState`接受一个函数来解决问题.

```js
// 该函数第一个参数是state，第二个参数是props
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```



#### 4.2.2 State合并更新

&emsp;&emsp;假设说`state`有两个属性，那么分别set不会导致其中一个丢失。



### 4.3 单向数据流

&emsp;&emsp;组件间是“单向”通信的，父组件只能通过属性传入子组件数据。子组件不能改变 也不能访问任何的父组件数据。`state`类组件私有的，`props`是父级传入的





# 五、事件处理

&emsp;&emsp;React元素的时间处理和DOM元素的两点不同：

- React的时间命名采用小驼峰式（camelCase），而不是纯小写。
- 使用JSX语法时需要传入一个函数作为时间处理函数。



### 5.1 阻止默认事件

&emsp;&emsp;React不能通过`return false`阻止默认事件。

**传统写法：**

```js
<a href="#" onClick="console.log('The link was clicked.'); return false">
    点我
</a>
```

**React写法：**

```js
function ActionLink() {
    function handleClick(e) {
        e.preventDefault();
        console.log('The link was clicked.');
    }
    return (
        <a href="#" onClick={handleClick}>
            点我
        </a>
    );
}
```

`e`：是一个合成事件，如果想了解更多，请查看 [SyntheticEvent](https://react-1251415695.cos-website.ap-chengdu.myqcloud.com/docs/events.html) 参考指南。



### 5.2 为this绑定类中方法

```js
class Toggle extends React.Component {
    constructor(props) {
        super(props);
        this.state = {isToggleOn: true};

        // 为了在回调中使用 `this`，这个绑定是必不可少的
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState(state => ({
            isToggleOn: !state.isToggleOn
        }));
    }

    render() {
        return (
            <button onClick={this.handleClick}>//必须将方法绑定this
                {this.state.isToggleOn ? 'ON' : 'OFF'}
            </button>
        );
    }
}

ReactDOM.render(
    <Toggle />,
    document.getElementById('root')
);
```

> 在`JSX`回调函数（例如：onClick）中`class`的方法默认不会绑定`this`。



#### 5.2.1 public Class fields & 箭头函数解决绑定问题

**public Class fields：**【推荐使用】

```js
class LoggingButton extends React.Component {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    // 注意: 这是 *实验性* 语法。
    handleClick = () => {
        console.log('this is:', this);
    }
    render() {
        return (
            <button onClick={this.handleClick}>
                Click me
            </button>
        );
    }
}
```

**箭头函数解决:**【会在每次渲染时创建不同的回调函数，如果该函数作为prop传入子组件可能会发生额外的重新渲染】

```js
class LoggingButton extends React.Component {
    handleClick() {
        console.log('this is:', this);
    }
    render() {
        // 此语法确保 `handleClick` 内的 `this` 已被绑定。
        return (
            <button onClick={(e) => this.handleClick(e)}>
                Click me
            </button>
        );
    }
}
```



### 5.3 向事件处理程序传递参数

```html
<!-- 箭头函数方式：e作为最后一个参数传入 -->
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<!-- 绑定this方式：this作为第一个参数传入 -->
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```



# 六、条件渲染

&emsp;&emsp;在`React`中，可以创建不同的组件来封装各种需要的行为，**依据应用的不同状态渲染部分内容。**【通过`if-else`或者三目】

```js
function Longin(props) {
    return <h1>登陆  注册</h1>
}

function Out(props) {
    return <h1>退出登录</h1>
}

function Greeting(props) {
    const flag = props.isLoginedIn;
    if (flag) {
        return <Longin/>;
    }
    return <Out />
}
ReactDOM.render(
    <Greeting isLoginedIn={true} />,
    document.getElementById("root"));
```



**条件渲染结合事件：**

```js
function Longin(props) {
    return <button onClick={props.onClick}>登陆</button>
}

function Out(props) {
    return <button onClick={props.onClick}>退出</button>
}

class Greeting extends React.Component{
    constructor(props) {
        super(props);
        //绑定
        this.setLogin = this.setLogin.bind(this);
        this.setOut = this.setOut.bind(this)

        this.state = {
            flag: false
        };
    }

    setOut() {//点击登录，显示退出
        this.setState({flag: false});
    };

    setLogin(){//点击退出，显示登陆
        this.setState({flag:true});
    };

    render() {
        const flag = this.state.flag;
        if (flag) {
            return <Longin onClick={this.setOut}/>;
        }else{
            return <Out onClick={this.setLogin} />
        }
    }
}

ReactDOM.render(
    <Greeting />,
    document.getElementById("root"));
```



### 6.1 JS中与 “&&”操作

> 在JS中`true && expression`会返回`expression`,而 `false && expression` 总是会返回 `false`。

&emsp;&emsp;因此，如果条件是 `true`，`&&` 右侧的元素就会被渲染，如果是 `false`，React 会忽略并跳过它。

```js
function Mailbox(props) {
    const unreadMessages = props.unreadMessages;
    return (
        <div>
            <h1>你好!</h1>
            {unreadMessages.length > 0 &&
            <h2>
                你这个数组有 {unreadMessages.length} 个内容！
            </h2>
            }
        </div>
    );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
    <Mailbox unreadMessages={messages} />,
    document.getElementById('root')
);
```



### 6.2 三目运算符

&emsp;&emsp;这个很熟悉了，不记录。



### 6.3 阻止组件渲染

&emsp;&emsp;在极少数情况下可能希望能隐藏组件，即使他已经被其他组件渲染。此时可以让`render`方法直接返回`null`从而不尽兴任何渲染。

&emsp;&emsp;当`render`返回为`null`时并不会影响组件的生命周期。



# 七、列表 & key



### 7.1 渲染多个组件

&emsp;&emsp;使用`JSX`在`{}`内 构建一个元素集合。

```js
const numbers = [1, 2, 3, 4, 5];
const ListItems = numbers.map((number) => <li>{number}</li>);

ReactDOM.render(
    <ul>{ListItems}</ul>,
    document.getElementById("root")
);
```



**将上边的例子重构为一个组件：**【功能：根据传入数组生成列表】

```js
function List2Ul(props) {
    const lis = props.arr.map((li) => <li key={li.toString()}>{li}</li>);
    //不加key会出现警告：a key should be provided for list items
    return (
        <ul>{lis}</ul>
    );
}

const numbers = ["wang", "yi", "ming", "4", "5"];
ReactDOM.render(
    <List2Ul arr={numbers}/>,
    document.getElementById("root")
);
```



### 7.2 key

&emsp;&emsp;`key`可以帮助React识别哪些元素改变了，比如添加修改，因此我们应该给数组中的每个元素赋予一个确定的标记`key`。通常我们使用`id`作为`key`是最理想的，当元素没有`id`时使用`index`。

```html
const todoItems = todos.map((todo) =>
    <li key={todo.id}>
			{todo.text}
		</li>
);
//=============================== 上面使用id，下面使用index==============================//
const todoItems = todos.map((todo, index) =>
    <li key={index}>
        {todo.text}
    </li>
);
```

> 如果列表项目的顺序可能发生变化时，不建议使用index来作为key。【性能会变差】



#### 7.2.1 正确的使用key

&emsp;&emsp;假设说我们提取出一个组件，当这个组件需要被循环使用时，我们应该在这个组件上加`key`。而不是组件内部或其他。

>一个好的经验法则是：在 `map()` 方法中的元素需要设置 key 属性。



#### 7.2.2 key知识在兄弟节点间必须为1

&emsp;&emsp;`key`只是在兄弟节点之间应该独一无二，不需要全局唯一。

> **注意：**`props.key`是读不出来的，可以使用其他关键字代替。



### 7.3 在JSX中嵌入map

```js
function NumberList(props) {
    const numbers = props.numbers;
    return (
        <ul>
            {numbers.map((number) =>
                <li key={number.toString()}>
                    {number}
                </li>
            )}
        </ul>
    );
}
```

> 这种方法让代码更加清晰，如果map嵌套太多那就应该提取一个组件。



# 八、表单

&emsp;&emsp;在`HTML`中，表单元素通常自己维护`state`，并根据用户输入进行更新。而`React`中**"可变状态"【数据】**通常保存在组件的`state`中，并且只能通过使用`setState`来更新。



### 8.1 受控组件

&emsp;&emsp;受控组件，把两者结合了起来，让`React`的`state`成为“唯一数据源”。渲染表单的`React`组件控制者用户输入过程中表单发生的操作。

#### 8.1.1 input标签

```js
class NameForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {value: ''};

        //绑定
        this.handleChange = this.handleChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }
    
    //输入数据后将数据存入state
    handleChange(event) {//event传入的是调用的DOM
        this.setState({
            value: event.target.value
        })
    }
    
    handleSubmit(event){
        alert('提交的名字：' + this.state.value);
        event.preventDefault();//阻止默认事件
    }
    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <label>
                    名字:
                    <input type="text" value={this.state.value} onChange={this.handleChange}/>
                </label>
                <input type="submit" value="提交"/>
            </form>
        );
    }
}
```

- `value`:绑定`this.state.value`，其显示的始终是`state`中的`value`
- `handlechange`：每次按键都会更新`state.value`.【因此显示的值会随着用户输入而更新】

> 此种方式的优势就是可以对输入的参数进行统一处理。



#### 8.1.2 textarea标签

&emsp;&emsp;和`input`标签非常相似。

```js
class NameForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {value: ''};

        //绑定
        this.handleChange = this.handleChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }
    
    //输入数据后将数据存入state
    handleChange(event) {//event传入的是调用的DOM
        this.setState({
            value: event.target.value
        })
    }
    
    handleSubmit(event){
        alert('提交的文章：' + this.state.value);
        event.preventDefault();//阻止默认事件
    }
    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <label>
                    文章:
                    <textarea value={this.state.value} onChange={this.handleChange}/>
                </label>
                <input type="submit" value="提交"/>
            </form>
        );
    }
}
```



#### 8.1.3 select标签

&emsp;&emsp;在React中并不使用`selected`进行默认选中，而是直接在`select`标签上使用`value`属性。

```js
class NameForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {value: 'coconut'};//在这里配置默认

        //绑定
        this.handleChange = this.handleChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }
    
    //输入数据后将数据存入state
    handleChange(event) {//event传入的是调用的DOM
        this.setState({
            value: event.target.value
        })
    }
    
    handleSubmit(event){
        alert('您喜欢的风味是：' + this.state.value);
        event.preventDefault();//阻止默认事件
    }
    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <label>
                    请选择你喜欢的风味:
                    <select value={this.state.value} onChange={this.handleChange}>
                        <option value="grapefruit">葡萄柚</option>
                        <option value="lime">酸橙</option>
                        <option value="coconut">椰子</option>
                        <option value="mango">芒果</option>
                    </select>
                </label>
                <input type="submit" value="提交"/>
            </form>
        );
    }
}
```

> 我们也可以将数组传入value，以支持选择多个选项



#### 8.1.4 文件input标签

&emsp;&emsp;因为value是只读的，所以它是一个`React`中的一个非受控组件，暂无讲解。



#### 8.1.5 处理多个输入

&emsp;&emsp;当需要处理多个`input`元素时，可以给每个元素添加`name`属性，并让处理函数根据 `event.target.name` 的值选择要执行的操作。

```js
class Reservation extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            isGoing: true,
            numberOfGuests: 2
        };
        this.handleInputChange = this.handleInputChange.bind(this);
    }

    handleInputChange(event) {
        const target = event.target;
        const value = target.type === 'checkbox' ? target.checked : target.value;
        const name = target.name;

        this.setState({
            [name]: value //使用变量作为属性名
        });
    }
  
    render() {
        return (
            <form>
                <label>
                    参与:
                    <input
                        name="isGoing"
                        type="checkbox"
                        checked={this.state.isGoing}
                        onChange={this.handleInputChange}
                    />
                </label>
                <label>
                    来宾人数:
                    <input
                        name="numberOfGuests"
                        type="number"
                        value={this.state.numberOfGuests}
                        onChange={this.handleInputChange}
                    />
                </label>
            </form>
        );
    }
}
```



#### 8.1.6 受控输入空值

&emsp;&emsp;此处没看懂



### 8.2 成熟的解决方案

&emsp;&emsp;Formik](https://jaredpalmer.com/formik)，包含验证、追踪访问字段以及处理表单提交的完整解决方案。建立在受控组件和管理 state 的基础上。



### 8.3 受控组件的替代品

&emsp;&emsp;[非受控组件](https://react-1251415695.cos-website.ap-chengdu.myqcloud.com/docs/uncontrolled-components.html), 这是实现输入表单的另一种方式。





# 九、状态提升

&emsp;&emsp;通常**多个组件需要反映相同的变化数据**，**这时建议将共享状态提升到最近的父组件中去。**



**判断水是否达到100度的组件：**

```js
/**
 * 判断是否煮沸
 */
function BoilingVerdict(props) {
    if (props.celsius >= 100) {
        return <p>水开了，倒水吧</p>
    }
    return <p>水还没有开</p>
}

/**
 * 渲染一个input用于输入温度
 */
class Calculator extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            temperature: ''//温度
        };
        this.handleChange = this.handleChange.bind(this);
    }

    /**
     * 输入事件
     */
    handleChange(event) {
        this.setState({
            temperature: event.target.value
        })
    }

    render() {
        const temperature = this.state.temperature;//温度
        return (
            <fieldset>
                <legend>
                    以摄氏度输入温度：
                </legend>
                <input
                    value={temperature}
                    onChange={this.handleChange}
                />
                <BoilingVerdict celsius={parseFloat(temperature)}/>
            </fieldset>
        );
    }
}
```



**改进为可以输入华摄氏度和摄氏度：【状态提升】：**

```js
/**
 * 判断是否煮沸
 */
function BoilingVerdict(props) {
    if (props.celsius >= 100) {
        return <p>水开了，倒水吧</p>
    }
    return <p>水还没有开</p>
}

/**
 * 判断该温度是否水开
 */
class Calculator extends React.Component {
    constructor(props) {
        super(props);

        this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
        this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
        this.state = {
            temperature:'',
            scale: 'c'
        };
    };
    /**
     * 处理摄氏度
     */
    handleCelsiusChange(temperature) {
        this.setState({scale: 'c', temperature});
    }

    /**
     * 处理华摄氏度
     */
    handleFahrenheitChange(temperature) {
        this.setState({
            scale: 'f',
            temperature
        });
    }

    render() {
        const scale = this.state.scale;
        const temperature = this.state.temperature;
        const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;//摄氏度
        const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;//华摄氏度

        return(
            <div>
                <TemperatureInput
                    scale="c"
                    temperature={celsius}
                    onTemperatureChange={this.handleCelsiusChange} />

                <TemperatureInput
                    scale="f"
                    temperature={fahrenheit}
                    onTemperatureChange={this.handleFahrenheitChange} />
                <BoilingVerdict
                    celsius={parseFloat(celsius)}
                />
            </div>
        )
    }
}


/**
 * 温度的名称
 */
const scaleNames={
    c: '摄氏度',
    f: '华摄氏度'
}


/**
 * 温度输入
 */
class TemperatureInput extends React.Component {
    constructor(props) {
        super(props);
        this.handleChange = this.handleChange.bind(this);
    };

    /**
     * 输入事件
     */
    handleChange(event) {
        this.props.onTemperatureChange(event.target.value);
    }


    render() {
        const temperature = this.props.temperature;
        const scale = this.props.scale;

        return(
            <fieldset>
                <legend>
                    请输入 {scaleNames[scale]}:
                </legend>
                <input value={temperature}
                    onChange={this.handleChange}
                />
            </fieldset>
        )
    }
}

/**
 * 转成摄氏度
 */
function toCelsius(fahrenheit) {
    return (fahrenheit - 32) * 5 / 9;
}

/**
 * 转成华摄氏度
 */
function toFahrenheit(celsius) {
    return (celsius * 9 / 5) + 32;
}

/**
 * 传入温度字符串和转换的方法，返回想要的结果
 */
function tryConvert(temperature, convert) {
    const input = parseFloat(temperature);
    if (Number.isNaN(input)) {
        return '';
    }
    const output = convert(input);
    const rounded = Math.round(output * 1000) / 1000;
    return rounded.toString();
}



ReactDOM.render(
    <Calculator />,
    document.getElementById("root")
)
```



### 学习小结：

> &emsp;&emsp;简单的来看React组件由类和函数的形式体现，使用ReactDOM.render来挂到根结点。state时组件的私有属性，props是传入的属性对象，组件间通信通过props传入函数的形式。状态提升就是兄弟组件通信的方式，说白了就是通过props方式的一种父子同心。



# 十、组合&继承



### 10.1 包含关系

```js
function FancyBorder(props) {
    return (
        <div className={'FancyBorder FancyBorder-' + props.color}>
            {props.children}
        </div>
    );
}

function WelcomeDialog() {
    return (
        <FancyBorder color="blue">
            {/* 从这里开始 */}
            <h1 className="Dialog-title">
                欢迎
            </h1>
            <p className="Dialog-message">
                请访问我们的。。。。。
            </p>
            {/* 到这里结束，都会由props.children传入子组件 */}
        </FancyBorder>
    );
}
```

&emsp;&emsp;对于无法提前知晓它们子组件的具体内容的组件，使用这种方式比较好。例如：侧边框和对话框。



**包含关系另一种使用方式：**

```js
function SplitPane(props) {
    return (
        <div className="SplitPane">
            <div className="SplitPane-left">
                {props.left}
            </div>
            <div className="SplitPane-right">
                {props.right}
            </div>
        </div>
    );
}

function App() {
    return (
        <SplitPane
            left={
                <Contacts />
            }
            right={
                <Chat />
            } />
    );
}
```

> 组件本质就是对象，所以可以这样传递。



### 10.2 组合

**定制的渲染组件：**

```js
function Dialog(props) {
    return (
        <FancyBorder color="blue">
            <h1 className="Dialog-title">
                {props.title}
            </h1>
            <p className="Dialog-message">
                {props.message}
            </p>
        </FancyBorder>
    );
}
function WelcomeDialog() {
    return (
        <Dialog
            title="欢迎"
            message="感谢您访问我们的太空船!" />

    );
}
```



**class形式的组合：**

```js
function Dialog(props) {
    return (
        <FancyBorder color="blue">
            <h1 className="Dialog-title">
                {props.title}
            </h1>
            <p className="Dialog-message">
                {props.message}
            </p>
            {props.children}
        </FancyBorder>
    );
}

class SignUpDialog extends React.Component {
    constructor(props) {
        super(props);
        this.handleChange = this.handleChange.bind(this);
        this.handleSignUp = this.handleSignUp.bind(this);
        this.state = {login: ''};
    }

    render() {
        return (
            <Dialog title="Mars Exploration Program"
                    message="How should we refer to you?">
                <input value={this.state.login}
                       onChange={this.handleChange} />

                <button onClick={this.handleSignUp}>
                    Sign Me Up!
                </button>
            </Dialog>
        );
    }

    handleChange(e) {
        this.setState({login: e.target.value});
    }

    handleSignUp() {
        alert(`Welcome aboard, ${this.state.login}!`);
    }
}
```



# 十一、React哲学【设计】



### 11.1 将设计好的UI划分为组件层次

&emsp;&emsp;一个组件原则上只能负责一个功能。如果它需要负责更多的功能，这时候就应该考虑将它拆分成更小的组件。当我们设计的比较恰当时。会于数据模型一一对应。



### 11.2 用React创建一个静态模版

&emsp;&emsp;我们最好将渲染 UI 和添加交互这两个过程分开，先设计一个不包含交互的UI。即使你已经熟悉了 *state* 的概念，也**完全不应该使用 state** 构建静态版本。state 代表了随时间会产生变化的数据，应当仅在实现交互时使用。所以构建应用的静态版本时，你不会用到它。



### 11.3 确实UI state的最小表示

&emsp;&emsp;

### 11.4 确定state防止的位置

&emsp;&emsp;

### 11.5 添加反向数据流

&emsp;&emsp;
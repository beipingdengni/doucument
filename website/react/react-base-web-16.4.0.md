## 地址依赖

```html
<script src="https://cdn.staticfile.org/react/16.4.0/umd/react.development.js"></script>
<script src="https://cdn.staticfile.org/react-dom/16.4.0/umd/react-dom.development.js"></script>
<!-- 生产环境中不建议使用 -->
<script src="https://cdn.staticfile.org/babel-standalone/6.26.0/babel.min.js"></script>

<!-- router -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-router/4.3.1/react-router.js"></script>
<script src="https://cdn.staticfile.org/react-router-dom/4.3.1/react-router-dom.min.js"></script>
```

## 渲染react

```react
 ReactDOM.render(
   <div>
     <h1>hello world</h1>
   </div>,
   document.getElementById("example")
 );
```

## 定义组件

### 函数组件

```react
const Dashboard = (props) => <div><h1>Dashboard APP  </h1> <App/> </div>;
```

### 类组件

```react
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      testVal: "测试输入，不用重复的渲染",
      date: new Date(),
    };

    // 绑定this才能操作
    // this.handleChange = this.handleChange.bind(this);
  }

  // 事件绑定了
  // handleChange(e) {
  //   this.setState({ testVal: e.target.value });
  // }

  // 以下方法可以忽略少调用， this.handleChange.bind(this)
  handleChange = (e) => {
    this.setState({ testVal: e.target.value });
  };

  // hook 函数
  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date(),
    });
  }

  render() {
    return (
      <div>
        <input
          name="a"
          type="text"
          value={this.state.testVal}
          onChange={this.handleChange}
          style={{ width: "500px", height: "50px" }}
          />
        <br />
        <div>
          <h1>{this.state.testVal}</h1>
        </div>
        <div>
          <h2>现在是 {this.state.date.toLocaleTimeString()}</h2>
        </div>
      </div>
    );
  }
}
```

## 路由

> 依赖组件： react-router、react-router-dom

```react
const {
  BrowserRouter, // 浏览器路由
  HashRouter, // 使用hash路由
  Route,    // 站位符号，路由
  IndexRoute,
  hashHistory,
  IndexLink,
  Link,
  browserHistory, // 存储路由历史
  Switch,    // 切换组件，内部只展示一个 Route
} = ReactRouterDOM;

// 组件回到顶部
class ScrollToTopOnMount extends React.Component {
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    window.scroll(0, 0);
    console.log("执行完成");
  }
  render() {
    return null;
  }
}

const A = (props) => <h1>A</h1>;
const B = (props) => <h1>B</h1>;
const C = (props) => <h1>C</h1>;

const Dashboard = (props) => <div><h1>Dashboard APP  </h1> <App/> </div>;

class Home extends React.Component {
  constructor(props) {
    super(props);
    console.log(this.props);
  }
  // 回到主页
  goToHome = () => this.props.history.push(`${this.props.match.url}`);

  render() {
    return (
      <div>
        {
          // 回到顶部
          <ScrollToTopOnMount />
        }
        <h1 onClick={this.goToHome}>Homes</h1>
        <ul>
          <li>
            <Link to={`${this.props.match.url}/a`}>TO A</Link>
          </li>
          <li>
            <Link to={`${this.props.match.url}/b`}>TO B</Link>
          </li>
          <li>
            <Link to={`${this.props.match.url}/c`}>TO C</Link>
          </li>
        </ul>
        <Switch>
          <Route
            path={`${this.props.match.url}`}
            exact
            component={Dashboard}
            />
          <Route path={`${this.props.match.url}/a`} exact component={A} />
          <Route path={`${this.props.match.url}/b`} exact component={B} />
          <Route path={`${this.props.match.url}/c`} exact component={C} />
          <Route
            path="/**"
            component={(props) => <h1>not found</h1>}
            ></Route>
        </Switch>
      </div>
    );
  }
}

ReactDOM.render(
  <HashRouter history={hashHistory}>
    <Switch>
      <Route path="/" exact component={(props) => <h1>login</h1>} />
      <Route path="/home" component={(props) => <Home {...props} />} />
      <Route path="/**" component={(props) => <h1>not found</h1>}></Route>
    </Switch>
  </HashRouter>,
  document.getElementById("example")
);
```



## 使用CDN加载实现react（完成代码）

```jsx
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <script src="https://cdn.staticfile.org/react/16.4.0/umd/react.development.js"></script>
    <script src="https://cdn.staticfile.org/react-dom/16.4.0/umd/react-dom.development.js"></script>
    <!-- 生产环境中不建议使用 -->
    <script src="https://cdn.staticfile.org/babel-standalone/6.26.0/babel.min.js"></script>

    <!-- router -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react-router/4.3.1/react-router.js"></script>
    <script src="https://cdn.staticfile.org/react-router-dom/4.3.1/react-router-dom.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      class App extends React.Component {
        constructor(props) {
          super(props);

          this.state = {
            testVal: "测试输入，不用重复的渲染",
            date: new Date(),
          };

          // 绑定this才能操作
          // this.handleChange = this.handleChange.bind(this);
        }

        // 事件绑定了
        // handleChange(e) {
        //   this.setState({ testVal: e.target.value });
        // }

        // 以下方法可以忽略少调用， this.handleChange.bind(this)
        handleChange = (e) => {
          this.setState({ testVal: e.target.value });
        };

        // hook 函数
        componentDidMount() {
          this.timerID = setInterval(() => this.tick(), 1000);
        }

        componentWillUnmount() {
          clearInterval(this.timerID);
        }

        tick() {
          this.setState({
            date: new Date(),
          });
        }

        render() {
          return (
            <div>
              <input
                name="a"
                type="text"
                value={this.state.testVal}
                onChange={this.handleChange}
                style={{ width: "500px", height: "50px" }}
              />
              <br />
              <div>
                <h1>{this.state.testVal}</h1>
              </div>
              <div>
                <h2>现在是 {this.state.date.toLocaleTimeString()}</h2>
              </div>
            </div>
          );
        }
      }

      const {
        BrowserRouter, // 浏览器路由
        HashRouter, // 使用hash路由
        Route,
        IndexRoute,
        hashHistory,
        IndexLink,
        Link,
        browserHistory, // 存储路由历史
        Switch,
      } = ReactRouterDOM;

      class ScrollToTopOnMount extends React.Component {
        constructor(props) {
          super(props);
        }

        componentDidMount() {
          window.scroll(0, 0);
          console.log("执行完成");
        }

        render() {
          return null;
        }
      }

      const A = (props) => <h1>A</h1>;
      const B = (props) => <h1>B</h1>;
      const C = (props) => <h1>C</h1>;

      const Dashboard = (props) => (
        <div>
          <h1>Dashboard APP </h1> <App />{" "}
        </div>
      );

      class Home extends React.Component {
        constructor(props) {
          super(props);
          console.log(this.props);
        }
        goToHome = () => this.props.history.push(`${this.props.match.url}`);

        render() {
          return (
            <div>
              {
                // 回到顶部
                <ScrollToTopOnMount />
              }
              <h1 onClick={this.goToHome}>Homes</h1>
              <ul>
                <li>
                  <Link to={`${this.props.match.url}/a`}>TO A</Link>
                </li>
                <li>
                  <Link to={`${this.props.match.url}/b`}>TO B</Link>
                </li>
                <li>
                  <Link to={`${this.props.match.url}/c`}>TO C</Link>
                </li>

                <li>
                  <Link to={`/`}>GO TO LOGIN</Link>
                </li>
              </ul>
              <Switch>
                <Route
                  path={`${this.props.match.url}`}
                  exact
                  component={Dashboard}
                />
                <Route path={`${this.props.match.url}/a`} exact component={A} />
                <Route path={`${this.props.match.url}/b`} exact component={B} />
                <Route path={`${this.props.match.url}/c`} exact component={C} />
                <Route
                  path="/**"
                  component={(props) => <h1>not found</h1>}
                ></Route>
              </Switch>
            </div>
          );
        }
      }

      // this.props.history.push({path: "/page1", query:{/*要传的参数以对象的键值对形式放在这里*/ name:"zsk"}});
      //console.log(this.props.match.params.id) //这种是通过路径上面的:id传过来的参数
      //console.log(this.props.location.query) //这是通过push的参数对象中的query传过来的 和vue的query有区别 它不在地址栏 刷新丢失
      //console.log(this.props.location.state) //这是通过push的参数对象中的state传过来的 它不在地址栏 刷新丢失
      //console.log(this.props.location.search) //暴露在地址栏，需要自行处理获取数据
      // Switch 内部只展示一个路由

      ReactDOM.render(
        // <App name="APP Main DAY 01" />,
        <HashRouter history={hashHistory}>
          <Switch>
            <Route
              path="/"
              exact
              component={(props) => (
                <h1 onClick={(e) => {  props.history.push(`/home`);}} >
                  login
                </h1>
              )}
            />
            <Route path="/home" component={(props) => <Home {...props} />} />
            <Route path="/**" component={(props) => <h1>not found</h1>}></Route>
          </Switch>
        </HashRouter>,
        document.getElementById("example")
      );
    </script>
    <!-- <script type="text/babel" src="helloworld_react.js"></script> -->
  </body>
</html>
```


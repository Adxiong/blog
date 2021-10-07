---
layout: post
title: React-router实现路由集中管理
date: 2021-05-21
Author: 来自Adxiong
tags: [React, javascript, React-router]
comments: true
---
由于之前一直使用vue开发，对vue-router的集中式管理甚是喜欢，而在react中，为了避免路由到处引入，不便管理，我打算对route进行集中管理，创建。方便日后处理鉴权。
<!-- more -->

# 1.创建Router文件夹
在index.js中集中引入Component（也可以利用react.lazy进行懒加载引入）
```js
import Home from"../views/Home";
import Exercise from'../views/Exercise';
import Enter from"../views/Exercise/Enter";
import Details from "../views/Exercise/Details";
import Profile from'../views/Profile';
const Routes = [
  {
    path:"/home",
    name:"Home",
    title:"主页",
    component:Home
  },
  {
    path:"/exercise",
    name:"Exercise",
    title:"练习",
    component:Exercise,
    children:[
      {
        path:"/exercise/enter",
        name:"Enter",
        title:"路口",
        component:Enter
      },
      {
        path:"/exercise/details",
        name:"Details",
        title:"答题",
        component:Details
      }
    ]
  },
  {
    path:"/profile",
    name:"Profile",
    title:"练习",
    component:Profile
  }
]

export default Routes;

```

# 2.创建Utils公用文件
这里实现一个创建route的函数 ,引入之前的路由文件，对其遍历创建Route,在这里可以根据自己的需求添加一些鉴权内容。第一个参数是routes数组，第二个是重定向的path。
```js
import {Switch,Route ,Redirect } from "react-router-dom";

const MRoutes = (routes,redirectRoute)=>{
  return(
    <Switch>
      {
        routes.map((route)=>
            <Route key={route.name} path={route.path} render={props=>
              route.render?(
                route.render({...props , route:route.children})
              ):(
                <route.component {...props} route={route.children}/>
              )
            }></Route>
        )
      }
      <Redirect to={redirectRoute}></Redirect>
    </Switch>
  )
}
export default  MRoutes;
```

# 3.第一层路由渲染
```js
import Routes from "./Router";
import MRoutes from "./utils/createRoute";
import Header from "./components/Header";
import Login from "./components/Login";
import './App.css';

function App() {
  return (
    <div className="App">
      <Header></Header>
      {
        MRoutes(Routes,"/home")
      }
      <Login></Login>
    </div>
  );
}

export default App;

```
# 4.子路由渲染
```js
import MRoutes from "../../utils/createRoute";
import "./index.less";

const exercise = (props)=>{
  return(
    <div className="exercise">
      {
        MRoutes(props.route ,  "/exercise/enter")
      }
    </div>
  )
}

export default exercise;
```


总结：把每个route的children节点传递出去，在对应层级界面就可以看见自己的子路由信息，然后调用我们刚封装的创建路由函数就可以创建。函数中可以传递一个string参数,作为默认的重定向path
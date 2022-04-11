# umi使用指南

**umi全局安装**

```shell
yarn add global umi
```

如果安装完成后，输入`umi -v` 输入umi命令找不到，问题是 yarn的脚本没有被配置到环境变量中

**解决方法**:

1. 找到yarn运行命令的脚本位置

```shell
yarn global bin
```

![image-20220407175441132](https://gitee.com/zengshixin-dev/md-img/raw/master/img/image-20220407175441132.png)

2. 将该脚本地址添加到全局变量中去

![image-20220407175705583](https://gitee.com/zengshixin-dev/md-img/raw/master/img/image-20220407175705583.png)



## umi路由的两种配置方式

### **约定式路由**：

- umi 约定，src/pages文件夹中存放的是页面

- umi约定，页面的文件名，已经页面的文件路径，是该页面配置的路由

- umi约定，如果页面的文件名是index，则可以省略文件名（首页）(注意避免文件名和当前目录中的文件夹名称相同)

- umi约定，如果src/layout目录存在，那么该目录中的index.js表示全局通用的组件，布局中的children则会渲染具体的页面内容

- umi约定，如果pages中包含_layout.js, 那么 _layout.js所有在目录及其所有的子目录中的页面，都会公用该布局

- umi约定， 404约定，pages/404.js,表示404页面，该页面会在路由无匹配的情况下渲染。该约定在开发模式下不会渲染，在部署后会生效

- umi约定，以`[]`包裹的文件名是动态路由。 比如：

  ```bash
  └── pages
      └── [post].tsx
  ```

- umi约定，以`[$]`包裹的文件名是动态可选路由。 比如：

  ```shell
  └── pages
    └── [post$].tsx

**路由跳转**：

- 跳转链接： 导入`import {Link} from 'umi'`

  ```js
  import { Link } from 'umi';
  
  export default () => (
    <Link to="/list">Go to list page</Link>
  );
  ```

  

- 代码跳转： 导入`import { history } from 'umi'`

  ```js
  import { history } from 'umi';
  
  function goToListPage() {
    history.push('/list');
  }
  ```

> 导入模块时，@表示src目录

**路由信息获取**：

所有的**页面、布局组件**，都会通过属性，收到下面的属性

- match：等同于react-router的match
- history：等同于react-router的history（history.location.query被封装成了一个对象，使用的是query-string库进行的封装）
- location：等同于react-router的location（location.query被封装成了一个对象，使用的是query-string库进行的封装）
- route：对应的是路由配置

如果需要在普通组件中获取路由信息，则需要使用withRouter封装，可以通过`import {withRouter} from 'umi'`导入

### **配置式路由**：

> 配置式路由使用的了第三方库react-router-config, 当使用了配置式路由后，约定式路由便会失效

**配置方式**

- 在项目的根目录中的`.umirc.js`中书写配置,

  - component组件定义需要渲染的组件，的路径是相对src/pages的
  - 如果需要定义公共的组件内容，那么只需要在组件中增加routes属性即可，其他配置相同
  - 如果使用的组件需要经过其他组件进行处理（其实就是高阶组件），那么使用wrappers定义需要修饰的组件数组
    - 执行流程是将组件放进修饰的数组组件中的props.children的，执行流程是从数组后到前
  - 除了基本配置外，我们还可以增加额外的配置内容：比如添加一个age属性, 该属性会在配置的组件的route属性中
  - ![image-20220409133716640](https://gitee.com/zengshixin-dev/md-img/raw/master/img/image-20220409133716640.png)

  > .umirc.js

  ```js
  export default {
      routes: [
          {
              path: '/',
              component: '@/layouts',
              exact: false，
              // 定义公共的组件内容
              routes: [
                      {
                          path: '/',
                          component: './',
               			wrappers: ['@/hoc/privateCmp', '@/hoc/handlerTitle'],
                      	title: '首页'，
      					age: 123
                      },
                      {
                          path: '/login',
                          component: './login'
                      },
                      {
                          path: '/guide',
                          component: './guide'
                      }
              ]
          }
      ]
  }
  ```

  

在约定式路由中实现相关属性的扩展，

- 扩展路由属性（直接给函数对象增加属性即可）

  ```js
  function HomePage() {
    return <h1>Home Page</h1>;
  }
  
  HomePage.title = 'Home Page';
  
  export default HomePage;
  ```

  

- 页面通过高阶组件包裹来添加功能（给函数对象的wrappers属性，定义修饰的组件）

```js
import React from 'react'

function User() {
  return <>user profile</>
}

User.wrappers = ['@/wrappers/auth']

export default User
```


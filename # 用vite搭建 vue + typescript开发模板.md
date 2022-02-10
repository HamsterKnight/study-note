# 用vite搭建 vue + typescript开发模板

创建vue模板

```shell
yarn create vite <project-name> --template vue
```

安装库typescript, vue-tsc

```shell
yarn add  -D typescript vue-tsc
```

增加typescript配置
```shell
npx tsc --init
```
对tsconfig.json文件进行修改
```js
 "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
 "exclude": ["node_modules"]
```



src目录下增加识别.vue文件结尾的配置文件***env.d.ts***,文件内容如下
```js
/// <reference types="vite/client" />

declare module '*.vue' {
  import { DefineComponent } from 'vue'
  // eslint-disable-next-line @typescript-eslint/no-explicit-any, @typescript-eslint/ban-types
  const component: DefineComponent<{}, {}, any>
  export default component
}
```

修改package.json的build命令为下面的命令，才能对.vue文件进行类型检查，不输出生成文件
```shell
"build": "vue-tsc --noEmit && vite build",
```

将其他.js文件结尾的文件，修改为.ts结尾的文件

**以上配置参考**<https://cn.vitejs.dev/guide/features.html#hot-module-replacement>



### 配置全局共享样式

```

 css: {
    // 全局共享样式
    preprocessorOptions: {
      scss: {
        additionalData: '@import "@/style/base.scss";'
      }
    }
  }
 
这段代码的意思，应该是在所有的scss文件前都加入base.scss,所以是先要有引入相关的scss文件后，才会生效
```





### 配置eslint，对typescript及vue文件内容进行约束
```shell
npm install --save-dev eslint typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin
```
相关库的作用
- **eslint** : EsLint的核心代码
-  **typescript** : typescript代码
-  **@typescript-eslint/parser**： 让Eslint识别ts代码的核心库
-  **@typescript-eslint/eslint-plugin** 一个ESLint插件，包含了各类定义好的检测Typescript代码的规范
[如何用eslint约束typescript](https://typescript-eslint.io/docs/linting/)

#### 创建.eslintrc.js文件对eslint进行配置
```js
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
  ],
};
```

### 对vue文件进行esLint检查[配置参考文档]<https://eslint.vuejs.org/>
```
yarn add -D eslint-plugin-vue
```
修改上面的配置文件为
```
module.exports = {
  root: true,
  - //parser: '@typescript-eslint/parser',
  + "parser": "vue-eslint-parser",
  + parseOptions: {
  +   "parser": "@typescript-eslint/parser",
  +    "sourceType": "module"
  + },
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
  -  //'eslint:recommended',
  + 'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
  ],
};
```

### 配置代码美化
```shell
yarn add -D prettier eslint-config-prettier eslint-plugin-prettier
```
- prettier 代码美化核心库（如果不和eslint结合使用是需要安装vscode插件的，事实上eslint内部内置了prettier，不用安装vscode插件）
- eslint-config-prettier： 当eslint的配置和prettier的配置冲突时，以prettier的为准
- eslint-plugin-prettier:  eslint使用prettier的配置来进行代码检查

### 修改.eslintrc.js配置
```js
module.exports = {
  root: true,
  //parser: '@typescript-eslint/parser',
  "parser": "vue-eslint-parser",
  parserOptions : {
    "parser": "@typescript-eslint/parser",
    "sourceType": "module"
  },
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
  + 'plugin:prettier/recommended' // 需要放到最后
  ],
  rules:{
    "@typescript-eslint/no-explicit-any": "off",
    "@typescript-eslint/ban-types": "off",
  }
};
```
[eslint-config-prettier文档参照]<https://github.com/prettier/eslint-config-prettier>
[eslint-plugin-prettier配置文档]<https://github.com/prettier/eslint-plugin-prettier>

### package.json中增加运行脚本
```shell
    "lint": "eslint src",// eslint检查
    "lint:fix": "eslint --fix --ext .ts,.tsx,.vue src" //eslint问题修复，及代码格式化
```





### 代码提交规范  husky + commitlint + lint-staged

# ant-design-debug

调试 Ant Design 源码

- 参考: [为什么说 90% 的前端不会调试 Ant Design 源码？](https://mp.weixin.qq.com/s/0X3QNLgbqpk6jMjsAX3Jyw)

## 步骤

```bash
# 第一步
# 新建项目
yarn create react-app antd-react-test
# or
yarn create react-app .

npm start
# http://localhost:3000/

# 第二步
yarn add antd
# App.js 引入 Button 组件

# 打断点，vscode 启动 launch.json
# renderWithHooks 处，添加条件断点
#   var children = Component(props, secondArg);
# 表达式: Component.name.includes('Button')

# InternalButton 就是 antd 里的 Button 组件。

# clone antd
# --single-branch 是下载单个分支， --depth=1 是下载单个 commit
git clone --depth=1 --single-branch https://github.com/ant-design/ant-design.git

# 构建 antd dist
npm run di


st
```

antd package.json 中有三种入口, 分别对应了 lib、es、dist 的目录。

```json
{
  "main": "dist/antd.js",
  "module": "es/index.js",
  "unpkg": "dist/antd.min.js",
}
```

- main 是 commonjs 的入口，也就是 `require('antd')` 的时候会走这个。
- module 是 esm 的入口，也就是 `import xx from 'antd'` 的时候会走这个。
- unpkg 是 UMD 的入口，也就是通过 script 标签引入的时候或者 commonjs 的方式等都可以用。

所以 antd 项目里的 dist 命令就是单独生成 UMD 代码的，而 build 命令是生成这三种代码。

这三种形式的代码都是可用的，这里我们选择构建 UMD 形式的代码，因为它会用 webpack 打包


```bash
            tsc               babel              webpack
Button.tsx =====> Button.js ========> Button.js ========> bundle.js

# 1. webpack 的 sourcemap 默认只会根据最后一个 loader 的 sourcemap 来生成。
# 2. 所以使用 antd/dist/antd 的 sourcemap 只能关联到 babel 处理之前的代码，像 ts 语法、jsx 代码这些都没有了。
# 3. 还需要关联更上一级的 ts-loader 的 sourcemap
# 想映射回最初的 tsx 源码，只要关联了每一级 loader 的 sourcemap 就可以了。而这个是可以配置的，就是 devtool。
# 想要 sourcemap 映射到 tsx 源码，需要把 devtool 设置成 cheap-module-source-map，然后开启 babel-loader 和 ts-loader 的 sourcemap。

# sourceMap 解决方案: `node_modules/@ant-design/tools/lib/getWebpackConfig.js`
# 1. 新增配置 `babelConfig.sourceMap = true;` 开启 babel 的 sourceMap
# 2. `devtool: 'source-map',` 改为 `devtool: 'cheap-module-source-map',` 支持关联 loader
# 3. ts 也要生成 sourceMap, 在 antd 根目录的 tsconfig.json里改 `"sourceMap": true,`

# 重新编译 dist，将 dist 覆盖原依赖文件 node_modules/antd/dist
npm run dist

# 最后一步，要清除项目的 babel-loader 缓存 `node_modules/.cache/babel-loader`
# 重新debug，已经可以调试 antd 源码了
```

参考代码

- https://babeljs.io/docs/en/options#sourcemaps

```ts
babelConfig.sourceMaps = true;
```

- https://github.com/ant-design/antd-tools/blob/826a3c96c93e407dbff1c04f02de866261af4157/lib/getWebpackConfig.js#L61-L62
  - 修改此处 `devtool`
- https://github.com/ant-design/antd-tools/blob/826a3c96c93e407dbff1c04f02de866261af4157/lib/getWebpackConfig.js#L108-L150
  - 此处的 rules 中 `babel-loader` 会使用到配置 `babelConfig.sourceMaps`

```ts
  const babelConfig = require('./getBabelCommonConfig')(modules || false);

  // 新增配置
  babelConfig.sourceMaps = true; // 推荐
  // babelConfig.sourceMap = true;

  const config = {
    devtool: 'source-map',
    // 改为
    devtool: 'cheap-module-source-map',

    output: {
      path: getProjectPath('./dist/'),
      filename: '[name].js',
    },
```


# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `yarn start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

### `yarn test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `yarn build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `yarn eject`

**Note: this is a one-way operation. Once you `eject`, you can't go back!**

If you aren't satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you're on your own.

You don't have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn't feel obligated to use this feature. However we understand that this tool wouldn't be useful if you couldn't customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `yarn build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)

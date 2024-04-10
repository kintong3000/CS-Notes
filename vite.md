### 构建工具的作用：

1. 模块化开发支持: 支持直接从node modules里引入代码 + 多种模块化支持

2. 处理代码兼容性:比如babel语法降级，less,ts 语法转换(不是构建工具做的，构建工具将这些语法对应的处理工具集成进来自动化处理)

3. 提高项目性能: 压缩文件，代码分割

4. 优化开发体验:

- 构建工具会帮你自动监听文件的变化，当文件变化以后自动帮你调用对应的集成工具进行重新打包，然后再浏览器重新运
  行(整个过程叫做热更新，hot replacement
- 开发服务器: 跨域的问题，用react-cli create-react-element vue-cli解决跨域的问题





### 开发服务器的启动对比



当冷启动开发服务器时，基于打包器的方式（webpack）启动必须优先抓取并构建你的整个应用（打包），然后才能提供服务。

![image-20231202162018893](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/image-20231202162018893.png)



Vite 将会使用 [esbuild](https://esbuild.github.io/) [预构建依赖](https://vitejs.cn/vite3-cn/guide/dep-pre-bundling.html)。Vite 以 [原生 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 方式提供源码。（不需要打包）

这实际上是让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。（按需加载）

![image-20231202162026966](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/image-20231202162026966.png)



vite是基于es modules的，侧重点不一样，webpack更多的关注兼容性，而vite关注浏览器端的开发体验。



### 依赖解析和预构建

在默认情况下，我们的esmaule去导入资源的时候，要么是绝对路径，要么是相对路径。

既然我们现在的最佳实践就是node modules，那为什么es官方在我们导入非绝对路径和非相对路径的资源的时候不默认帮我们 搜寻node modules呢?

假设浏览器做了这个事情，所有依赖都由http请求获得(依赖里面含依赖)，吃网络性能，慢。vite会帮我完成依赖解析（路径补全）



开发环境下

当使用esm时：

1. vite会先进行**预构建**，预构建它们可以提高页面加载速度，并将 CommonJS / UMD 转换为 ESM 格式，且将有许多内部模块的 ESM 依赖关系转换为单个模块，以提高后续页面加载性能。预构建这一步由 [esbuild](http://esbuild.github.io/) 执行，这使得 Vite 的冷启动时间比任何基于 JavaScript 的打包器都要快得多。

2. 并重写导入为合法的 URL，例如 `/node_modules/.vite/deps/my-dep.js?v=f3sf2ebd` 以便浏览器能够正确导入它们。



生产环境使用**rollup**打包






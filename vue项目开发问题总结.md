# vue项目开发中遇到的各类问题记录

## 一.  Vue-cli引入`Zepto.js`过程以及报错解决

  1. 按照引入Jquery的方式引入Zepto之后控制台报错如下：
  
      `TypeError: Cannot read property ‘createElement’ of undefined`

  2. 报错原因及解决方案: https://sebastianblade.com/how-to-import-unmodular-library-like-zepto/ 
  3. 需要下载先 `npm install –save-dev script-loader exports-loader`
  4. 使用loader模块化加载`Zepto.js` 
  5. `webpack.base.conf.js`修改配置

      ```
          {
                // ...
                module: {
                    rules: [
                            //...
                            {
                                test: require.resolve('zepto'),
                                loader: 'exports-loader?window.Zepto!script-loader'
                            }
                    ]
                }
          }
      ```
    6. 如果需要全局引入可以在`main.js`中
  
        ```
        import $ from 'zepto'
        ```
## 二.  Vue-cli打包后`vendor.js`体积过大
  ### 在一个vue项目打包的 vender.js 足足 1.2M
  1. 解决方法: 打包 `vender` 时不打包 `vue、vuex、vue-router、axios` 等，换用国内的 bootcdn 直接引入到 `index.html` 中

  2. 修改配置 在`webpack.base.conf.js` 修改配置 module.exports添加
        ```
          // 配置cdn引入的文件 告诉webpack不用打包哪些文件
          externals: {
            "echarts": "echarts",
            'vue': 'Vue',
            'vue-router': 'VueRouter',
            "moment": "moment",
            "weui":"weui"
          }
        ```
      此时的 `vender` 包会非常小，如果不够小还可以拆分其他的库，此时增加了请求的数量，但是远比加载一个 1.2M 的 `vender` 快的多
 3. 使用按需导入的方式导入模块的体积增大,合理使用

## 三. 首屏加载过慢

   1. +原因:当打包构建应用时，Javascript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。
    解决方案:  路由懒加载
        ```
        {
          path: '/SalesDetails',
          name: 'SalesDetails',
          component: (resolve) => {
            require(['../components/SalesDetails'], resolve)
          },
          meta : {
            title:'销售明细'
          }
        }
        ```
## 四. HTML5 History 模式
  1. +后台配置 `https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90`
  2.  +前端配置
      ```
      const router = new VueRouter({
        mode: 'history',
        base:'/wxbi-vue/',
        routes: [...]
      })
      ```
## 五. .vue单文件引入css问题
 1. +如果在单文件中通过`@import '../base.css'`方式引入的css  则为全局的样式的css style 设置 scoped 无效
## 六. 项目引入文件路径的配置
1. +`@`代表src这个目录
2. +`webpack.base.conf.js` 配置
      ```
        resolve: {
            extensions: ['.js', '.vue', '.json'],
            alias: {
              'vue$': 'vue/dist/vue.esm.js',
              '@': resolve('src'),
            }
          }
      ```
## 七.上传到远程仓库github 在down下来 运行项目报错
  + 在项目的根目录创建一个`.postcssrc.js`文件  在文件里添加
  ```
    // https://github.com/michael-ciniawsky/postcss-load-config
      module.exports = {
        "plugins": {
          "postcss-import": {},
          "postcss-url": {},
          // to edit target browsers: use "browserslist" field in package.json
          "autoprefixer": {}
        }
      }
  ```
## 八. 父组件修改子组件中的样式
  ### 问题描述: 
  在 `vue` 的项目开发中，我们有时会引用外部组件，包括 UI 组件（`ElementUI`、`Mint`、`Swiper`）。
  当 `<style>` 标签有 `scoped` 属性时，它的 `CSS` 只作用于当前组件中的元素。
  但是在父组件中添加 `scoped` 之后，父组件的样式将不会渗透到子组件中，所以在父组件中书写子组件的样式是无效果的。
  ### 解决方案: 
  #### 方法一: 去掉`scoped`属性 缺点可能会造成全局污染
  #### 方法二: 本地和全局样式的混搭

        <style>
        /* 全局样式 */
        </style>
        <style scoped>
        /* 本地样式 */
        </style> 
  #### 方法三:
  + 如果你希望 `scoped` 样式中的一个选择器能够作用得“更深”，例如影响子组件，你可以使用  "`>>>`"  操作符：
  ```
      <style scoped>
        .father >>> .son-active {
        /* ... 
          在这里添加样式
          .father 父组件的选择器
          .son-active 子组件要修改元素的选择器
        */
        }
      </style>
  ```
  + 优点: 利于维护,实现组件的复用性,推荐使用此方法
  + 注意: 有些像 `SASS` 之类的预处理器无法正确解析  "`>>>`" 。这种情况下你可以用  "`/deep/`"  操作符取而代之 —— 这是一个 "`>>>`" 的别名，同样可以正常工作。
  + 补充: 
  
    1、通过 `v-html` 创建的 DOM 内容不受作用域内的样式影响，但是你仍然可以通过深度作用选择器来为他们设置样式
    2、CSS 作用域不能代替 class
    3、在递归组件中小心使用后代选择器
## 移除严格模式
[babel-plugin-transform-remove-strict-mode](https://github.com/genify/babel-plugin-transform-remove-strict-mode)

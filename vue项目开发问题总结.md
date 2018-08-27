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
  1. 第三方的依赖库 外部引入模块(CDN)

  2. `webpack.base.conf.js` 修改配置 module.exports添加
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

## 三. 首屏加载过慢

   1. +原因:当打包构建应用时，Javascript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。
      路由懒加载
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
## 移除严格模式
[babel-plugin-transform-remove-strict-mode](https://github.com/genify/babel-plugin-transform-remove-strict-mode)

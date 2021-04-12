# vue-cli4 项目优化

>因为使用了cli4，很多目录结构不见了，而相关配置是放在vue.config.js里面的，因此在根目录，新建一个vue.config.js


## 1、将 productionSourceMap 设为 false

```javascript

  module.exports = {
    productionSourceMap: false
  }
```

## 2、图片压缩
> vue正常打包之后一些图片文件很大，使打包体积很大，通过image-webpack-loader插件可将大的图片进行压缩从而缩小打包体积

1. 先安装依赖：cnpm install image-webpack-loader --save-dev
2. 在vue.config.js中module.exports写入：
```javascript
  module.exports = {
    productionSourceMap: false,
    chainWebpack: config => {
        // ============压缩图片 start============
        config.module
            .rule('images')
            .use('image-webpack-loader')
            .loader('image-webpack-loader')
            .options({ bypassOnDebug: true })
            .end()
        // ============压缩图片 end============
    }
  }
```
## 3、cdn配置（可选）
1. 配置文件声明 
```javascript
  // 是否为生产环境
  const isProduction = process.env.NODE_ENV !== 'development'

  // 本地环境是否需要使用cdn
  const devNeedCdn = false

  // cdn链接
  const cdn = {
    // cdn：模块名称和模块作用域命名（对应window里面挂载的变量名称）
    externals: {
        vue: 'Vue',
        vuex: 'Vuex',
        'vue-router': 'VueRouter'
    },
    // cdn的css链接
    css: [],
    // cdn的js链接
    js: [
      'https://cdn.staticfile.org/vue/2.6.10/vue.min.js',
      'https://cdn.staticfile.org/vuex/3.0.1/vuex.min.js',
      'https://cdn.staticfile.org/vue-router/3.0.3/vue-router.min.js'
    ]
  }
```

2. 在vue.config.js module.exports chainWebpack中写入：
```javascript
  // ============注入cdn start============
  config.plugin('html').tap(args => {
      // 生产环境或本地需要cdn时，才注入cdn
      if (isProduction || devNeedCdn) args[0].cdn = cdn
      return args
  })
  // ============注入cdn start============
```

3. 在vue.config.js module.exports configureWebpack中写入：
```javascript
  configureWebpack: config => {
    // 用cdn方式引入，则构建时要忽略相关资源
    if (isProduction || devNeedCdn) config.externals = cdn.externals
  }
```
4. 当前配置的vue.config.js

```javascript
  // 是否为生产环境
  const isProduction = process.env.NODE_ENV !== 'development'

  // 本地环境是否需要使用cdn
  const devNeedCdn = false

  // cdn链接
  const cdn = {
    // cdn：模块名称和模块作用域命名（对应window里面挂载的变量名称）
    externals: {
        vue: 'Vue',
        vuex: 'Vuex',
        'vue-router': 'VueRouter'
    },
    // cdn的css链接
    css: [],
    // cdn的js链接
    js: [
      'https://cdn.staticfile.org/vue/2.6.10/vue.min.js',
      'https://cdn.staticfile.org/vuex/3.0.1/vuex.min.js',
      'https://cdn.staticfile.org/vue-router/3.0.3/vue-router.min.js'
    ]
  }

  module.exports = {
    productionSourceMap: false,
    chainWebpack: config => {
        // ============压缩图片 start============
        config.module
            .rule('images')
            .use('image-webpack-loader')
            .loader('image-webpack-loader')
            .options({ bypassOnDebug: true })
            .end()
        // ============压缩图片 end============

        // ============注入cdn start============
        config.plugin('html').tap(args => {
            // 生产环境或本地需要cdn时，才注入cdn
            if (isProduction || devNeedCdn) args[0].cdn = cdn
            return args
        })
        // ============注入cdn start============
    },
    configureWebpack: config => {
        // 用cdn方式引入，则构建时要忽略相关资源
        if (isProduction || devNeedCdn) config.externals = cdn.externals
    }
  }
```

5. 在public index.html 写入

```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8" />
      <meta http-equiv="X-UA-Compatible" content="IE=edge" />
      <meta name="viewport" content="width=device-width,initial-scale=1.0" />
      <link rel="icon" href="<%= BASE_URL %>favicon.ico" />
      <!-- 使用CDN的CSS文件 -->
      <% for (var i in htmlWebpackPlugin.options.cdn &&
      htmlWebpackPlugin.options.cdn.css) { %>
      <link
          href="<%= htmlWebpackPlugin.options.cdn.css[i] %>"
          rel="stylesheet"
      />
      <% } %>
      <!-- 使用CDN的CSS文件 -->
      <title>cli3_base</title>
    </head>
    <body>
      <noscript>
          <strong
              >We're sorry but cli3_base doesn't work properly without
              JavaScript enabled. Please enable it to continue.</strong
          >
      </noscript>
      <div id="app"></div>

      <!-- 使用CDN的JS文件 -->
      <% for (var i in htmlWebpackPlugin.options.cdn &&
      htmlWebpackPlugin.options.cdn.js) { %>
      <script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
      <% } %>
      <!-- 使用CDN的JS文件 -->

      <!-- built files will be auto injected -->
    </body>
  </html>
```

6. 重启项目npm run serve
7. 在src/router.js修改
将
```javascript
Vue.use(Router)
```
改为
```javascript
if (!window.VueRouter) Vue.use(Router)
```
8. 重新启动npm run serve即可，现在的配置是开发环境，在浏览器的Network JS里面是看不到的。若想查看，请将vue.config.js里面的

```javascript
// 本地环境是否需要使用cdn
const devNeedCdn = false
```
改为
```javascript
// 本地环境是否需要使用cdn
const devNeedCdn = true
```
然后再次重启npm run serve，然后浏览器查看Network JS

## 4、代码压缩
1. 安装依赖：cnpm i -D uglifyjs-webpack-plugin
2. 在vue.config.js 最上边引入依赖
```javascript
// 代码压缩
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
```
3. 在vue.config.js module.exports configureWebpack 里面新增
```javascript
// 生产环境相关配置
if (isProduction) {
    // 代码压缩
    config.plugins.push(
        new UglifyJsPlugin({
            uglifyOptions: {
                //生产环境自动删除console
                compress: {
                    warnings: false, // 若打包错误，则注释这行
                    drop_debugger: true,
                    drop_console: true,
                    pure_funcs: ['console.log']
                }
            },
            sourceMap: false,
            parallel: true
        })
    )
}

```

## 5、开启Gzip
> Gzip需要配合nginx使用

1. 安装依赖：cnpm install --save-dev compression-webpack-plugin
2. 在vue.config.js 顶部引入依赖
```javascript
// gzip压缩
const CompressionWebpackPlugin = require('compression-webpack-plugin')
```
3. 在vue.config.js module.exports configureWebpack 里面新增，直接放在代码压缩下边即可
```javascript
// 生产环境相关配置
if (isProduction) {
    // 代码压缩
    // ..................
    // gzip压缩
    const productionGzipExtensions = ['html', 'js', 'css']
    config.plugins.push(
        new CompressionWebpackPlugin({
            filename: '[path].gz[query]',
            algorithm: 'gzip',
            test: new RegExp(
                '\\.(' + productionGzipExtensions.join('|') + ')$'
            ),
            threshold: 10240, // 只有大小大于该值的资源会被处理 10240
            minRatio: 0.8, // 只有压缩率小于这个值的资源才会被处理
            deleteOriginalAssets: false // 删除原文件
        })
    )
}
```

>vue cli3/vue cli4开启gzip打包报错：TypeError: Cannot read property 'tapPromise' of undefined<br/>
> 多半是版本有问题,使用compression-webpack-plugin@5.x.x 即可

4. nginx需要配置
```json
  location / {
    root         /data; # 项目根目录
    index /index.html;
    try_files $uri $uri/ /index.html;
    gzip on;
    gzip_static on; # gzip_static是nginx对于静态文件的处理模块，该模块可以读取预先压缩的gz文件，这样可以减少每次请求进行gzip压缩的CPU资源消耗。
    gzip_min_length 1k;
    gzip_http_version 1.1;
    gzip_comp_level 9;
    gzip_types  text/css application/javascript application/json;
    add_header "Access-Control-Allow-Origin" "*";   
    add_header "Access-Control-Allow-Methods" "*";  # 允许请求方法
    add_header "Access-Control-Allow-Headers" "*";  # 允许请求的 header

    if ($request_method = "OPTIONS") {
        return 204;
    }
  }
```

> gzip 参数说明
```javascript
  # 开启压缩  
  gzip  on;   
  # 设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于2k的字节数，小于2k可能会越压越大。  
  gzip_min_length 2k;  
  # 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。  
  # 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。  
  gzip_buffers 4 16k;  
  #压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间  
  gzip_comp_level 5;  
  # 默认值: gzip_types text/html (默认不对js/css文件进行压缩)  
  # 压缩类型，匹配MIME类型进行压缩  
  # 不能用通配符 text/*  
  # (无论是否指定)text/html默认已经压缩   
  # 设置哪压缩种文本文件可参考 conf/mime.types  
  gzip_types text/plain application/x-javascript text/css application/xml;    
  # 值为1.0和1.1 代表是否压缩http协议1.0，选择1.0则1.0和1.1都可以压缩  
  gzip_http_version 1.0   
  # IE6及以下禁止压缩  
  gzip_disable "MSIE [1-6]\.";   
  # 默认值：off  
  # Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。  
  # off - 关闭所有的代理结果数据的压缩  
  # expired - 启用压缩，如果header头中包含 "Expires" 头信息  
  # no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息  
  # no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息  
  # private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息  
  # no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息  
  # no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息  
  # auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息  
  # any - 无条件启用压缩  
  gzip_proxied expired no-cache no-store private auth;  
  # 给CDN和代理服务器使用，针对相同url，可以根据头信息返回压缩和非压缩副本  
  gzip_vary on;
```

## 6、公共代码抽离
1. 在vue.config.js module.exports configureWebpack 里面新增，直接放在gzip压缩下边即可
```javascript
  // 公共代码抽离
config.optimization = {
    splitChunks: {
        cacheGroups: {
            vendor: {
                chunks: 'all',
                test: /node_modules/,
                name: 'vendor',
                minChunks: 1,
                maxInitialRequests: 5,
                minSize: 0,
                priority: 100
            },
            common: {
                chunks: 'all',
                test: /[\\/]src[\\/]js[\\/]/,
                name: 'common',
                minChunks: 2,
                maxInitialRequests: 5,
                minSize: 0,
                priority: 60
            },
            styles: {
                name: 'styles',
                test: /\.(sa|sc|c)ss$/,
                chunks: 'all',
                enforce: true
            },
            runtimeChunk: {
                name: 'manifest'
            }
        }
    }
}
```


## 完整的vue-cli配置
```javascript
  // 代码压缩
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

// gzip压缩
const CompressionWebpackPlugin = require('compression-webpack-plugin')

// 是否为生产环境
const isProduction = process.env.NODE_ENV !== 'development'

// 本地环境是否需要使用cdn
const devNeedCdn = true

// cdn链接
const cdn = {
    // cdn：模块名称和模块作用域命名（对应window里面挂载的变量名称）
    externals: {
        vue: 'Vue',
        vuex: 'Vuex',
        'vue-router': 'VueRouter'
    },
    // cdn的css链接
    css: [],
    // cdn的js链接
    js: [
        'https://cdn.staticfile.org/vue/2.6.10/vue.min.js',
        'https://cdn.staticfile.org/vuex/3.0.1/vuex.min.js',
        'https://cdn.staticfile.org/vue-router/3.0.3/vue-router.min.js'
    ]
}

module.exports = {
    productionSourceMap: false,
    chainWebpack: config => {
        // ============压缩图片 start============
        config.module
            .rule('images')
            .use('image-webpack-loader')
            .loader('image-webpack-loader')
            .options({ bypassOnDebug: true })
            .end()
        // ============压缩图片 end============

        // ============注入cdn start============
        config.plugin('html').tap(args => {
            // 生产环境或本地需要cdn时，才注入cdn
            if (isProduction || devNeedCdn) args[0].cdn = cdn
            return args
        })
        // ============注入cdn start============
    },
    configureWebpack: config => {
        // 用cdn方式引入，则构建时要忽略相关资源
        if (isProduction || devNeedCdn) config.externals = cdn.externals

        // 生产环境相关配置
        if (isProduction) {
            // 代码压缩
            config.plugins.push(
                new UglifyJsPlugin({
                    uglifyOptions: {
                        //生产环境自动删除console
                        compress: {
                            warnings: false, // 若打包错误，则注释这行
                            drop_debugger: true,
                            drop_console: true,
                            pure_funcs: ['console.log']
                        }
                    },
                    sourceMap: false,
                    parallel: true
                })
            )

            // gzip压缩
            const productionGzipExtensions = ['html', 'js', 'css']
            config.plugins.push(
                new CompressionWebpackPlugin({
                    filename: '[path].gz[query]',
                    algorithm: 'gzip',
                    test: new RegExp(
                        '\\.(' + productionGzipExtensions.join('|') + ')$'
                    ),
                    threshold: 10240, // 只有大小大于该值的资源会被处理 10240
                    minRatio: 0.8, // 只有压缩率小于这个值的资源才会被处理
                    deleteOriginalAssets: false // 删除原文件
                })
            )

            // 公共代码抽离
            config.optimization = {
                splitChunks: {
                    cacheGroups: {
                        vendor: {
                            chunks: 'all',
                            test: /node_modules/,
                            name: 'vendor',
                            minChunks: 1,
                            maxInitialRequests: 5,
                            minSize: 0,
                            priority: 100
                        },
                        common: {
                            chunks: 'all',
                            test: /[\\/]src[\\/]js[\\/]/,
                            name: 'common',
                            minChunks: 2,
                            maxInitialRequests: 5,
                            minSize: 0,
                            priority: 60
                        },
                        styles: {
                            name: 'styles',
                            test: /\.(sa|sc|c)ss$/,
                            chunks: 'all',
                            enforce: true
                        },
                        runtimeChunk: {
                            name: 'manifest'
                        }
                    }
                }
            }
        }
    }
}
```
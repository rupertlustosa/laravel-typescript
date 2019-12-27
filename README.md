<p align="center"><img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1566331377/laravel-logolockup-cmyk-red.svg" width="400"></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

## About Laravel

- composer require laravel/ui --dev
- php artisan ui bootstrap
- php artisan ui vue
- npm install
- npm install vue-class-component vue-router vuex ts-loader typescript uglifyjs-webpack-plugin mini-css-extract-plugin axios --save-dev

## Preparing to Typescript
1 - Modify laravel-mixconfig to compile the Typescript.
```
// webpack.mix.js
mix.ts('resources/assets/js/app.ts', 'public/js')
```

2 - Rename resources/js/app.js to resources/js/app.ts and: 
```
import "./bootstrap"
import Vue from "vue"
import ExampleComponent from "./components/ExampleComponent.vue"

Vue.component('example', ExampleComponent);

new Vue({
    el: '#app'
});
```

3 - Rename resources/js/bootstrap.js to resources/js/bootstrap.ts and: 
```
/**
 * We'll load jQuery and the Bootstrap jQuery plugin which provides support
 * for JavaScript based Bootstrap features such as modals and tabs. This
 * code may be modified to fit the specific needs of your application.
 */

import axios from 'axios';
import * as _ from 'lodash';
import jQuery from 'jquery';
import * as Popper from 'popper.js';
import 'bootstrap';

axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';


let token : HTMLMetaElement | null = document.head!.querySelector('meta[name="csrf-token"]');

if (token) {
    axios.defaults.headers.common['X-CSRF-TOKEN'] = token.content;
} else {
    console.error('CSRF token not found: https://laravel.com/docs/csrf#csrf-x-csrf-token');
}

/**
 * Echo exposes an expressive API for subscribing to channels and listening
 * for events that are broadcast by Laravel. Echo and event broadcasting
 * allows your team to easily build robust real-time web applications.
 */

// import Echo from 'laravel-echo';

// window.Pusher = require('pusher-js');

// window.Echo = new Echo({
//     broadcaster: 'pusher',
//     key: process.env.MIX_PUSHER_APP_KEY,
//     cluster: process.env.MIX_PUSHER_APP_CLUSTER,
//     encrypted: true
// });

```

4 - Create typings.d.ts file inside resources/js
```
declare module '*.vue' {
    import Vue from 'vue'
    export default Vue
}

declare module 'jquery';

declare module 'lodash';
```

5 - Create tsconfig.json in the root of your project with:
```
{
    "compilerOptions": {
        "target": "es5",
        "strict": true,
        "module": "es2015",
        "moduleResolution": "node",
        "experimentalDecorators": true,
        "skipLibCheck": true
    },
    "include": [
        "resources/js/**/*"
    ],
    "exclude": [
        "node_modules",
        "vendor"
    ]
}

```

6 - Create webpack.config.js in the root of your project with:
```
const path = require('path');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const autoprefixer = require('autoprefixer');
const webpack = require('webpack');


let env = process.env.NODE_ENV;
let isDev = env === 'development';

const WEBPACK_CONFIG = {
    mode: env,
    entry: {
        app: ['./resources/js/app.ts', './resources/sass/app.scss'],
    },
    output: {
        publicPath: './public',
        path: path.resolve(__dirname, 'public'),
        filename: 'js/[name].js',
        chunkFilename: 'js/chunks/app.js'

    },
    module: {
        rules: [{
            test: /\.tsx?$/,
            loader: 'ts-loader',
            options: { appendTsSuffixTo: [/\.vue$/] },
            exclude: /node_modules/,
        },
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.scss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    'postcss-loader',
                    'sass-loader'
                ],
                exclude: /node_modules/,
            },
            {
                test: /\.(ttf|eot|svg|woff|woff2)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: '/fonts',
                    publicPath: '/fonts'
                }
            }
        ],
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].css'
        }),
        new VueLoaderPlugin(),
        new webpack.LoaderOptionsPlugin({
            options: {
                postcss: [
                    autoprefixer()
                ]
            }
        })

    ],
    resolve: {
        extensions: ['.js', '.jsx', '.vue', '.ts', '.tsx'],
        alias: {
            vue$: 'vue/dist/vue.esm.js',
        },
    },
    optimization: {
        splitChunks: {
            chunks: 'async',
            minSize: 30000,
            maxSize: 0,
            minChunks: 1,
            maxAsyncRequests: 5,
            maxInitialRequests: 3,
            automaticNameDelimiter: '~',
            name: true,
            cacheGroups: {
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    priority: -10
                },
                default: {
                    minChunks: 2,
                    priority: -20,
                    reuseExistingChunk: true
                }
            }
        }
    }


}

if (!isDev) {
    WEBPACK_CONFIG.optimization = {
        minimizer: [
            new UglifyJsPlugin({
                cache: true,
                parallel: true,
                sourceMap: true
            }),
            new OptimizeCSSAssetsPlugin({})
        ]
    }

    WEBPACK_CONFIG.plugins.push(
        new webpack.DefinePlugin({
            'process.env': {
                NODE_ENV: '"production"'
            }
        })

    )
}

module.exports = WEBPACK_CONFIG;

```

7 - Create postcss.config.js in the root of your project with:
```
module.exports = {
    plugins: {
        'autoprefixer': {}
    }
}

```

8 - Adicione em seu template principal:
```
<meta name="csrf-token" content="{{ csrf_token() }}">
```


```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
```



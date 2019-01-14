# はじめに

Rust製のWebフレームワーク：[`Iron`](https://github.com/iron/iron)と[`Vue.js`](https://github.com/vuejs/vue)、[`Webpack`](https://github.com/webpack/webpack)を使って、SPAなWebサイトを[`Heoku`](https://jp.heroku.com/) へとデプロイするチュートリアルです

実際につくったものはこちらです。

[iron_vue](https://github.com/S-H-GAMELINKS/iron_vue)

# 前提条件

前提条件として、[`Rust`](https://www.rust-lang.org/) と [`Node.js`](https://nodejs.org/ja/) がインストールされている必要があります。

また、パッケージマネージャとして [`yarn`](https://yarnpkg.com/lang/ja/) を使用しています。そちらもインストールされている必要があります。

また、 [`Heroku`](https://jp.heroku.com/) のアカウントが既に作成されていることも必要です。

[`Rust`](https://www.rust-lang.org/)、[`Node.js`](https://nodejs.org/ja/)、[`yarn`](https://yarnpkg.com/lang/ja/) のインストールに関しては下記の記事などを参考にしてください。

[rust-guide-ja installing-rust.md](https://github.com/cakecatz/rust-guide-ja/blob/master/docs/installing-rust.md)

[Node.js ダウンロード](https://nodejs.org/ja/download/)

[yarnをインストールする](https://qiita.com/suisui654/items/1b89446e03991c7c2c3d)

# チュートリアル
## Hello World

まずは、`Hello World` と表示させてみましょう

### ひな型を作る

まずは、`cargo new iron_vue` でひな型を作成します

```shell_session
cargo new iron_vue
```

コマンドが終了後、`cd iron_vue` でディレクトリを移動します

### Ironのインストール

`Cargo.toml` に `iron = "0.6.0"` を追加します。

```toml:Cargo.toml
[package]
name = "iron_vue"
version = "0.1.0"
authors = ["S-H-GAMELINKS <gamelinks007@gmail.com>"]
edition = "2018"

[dependencies]
iron = "0.6.0"
```

次に `cargo run` を実行します

```shell_session
cargo run
```

これで [`Iron`](https://github.com/iron/iron) のインストールは終わりです

### Hello World

お好きなエディタで`src/main.rs`を開き、下記のように変更します

```rust:src/main.rs
extern crate iron;

use iron::prelude::*;
use iron::status;

fn main() {
    Iron::new(|_: &mut Request| {
        Ok(Response::with((status::Ok, "Hello world!")))
    }).http("localhost:3000").unwrap();
}
```

その後、`cargo run` を実行してビルドとローカルサーバーの起動を行います

```shell_session
cargo run
```

最後に、お好きなブラウザで`localhost:3000` にアクセスし、`Hello World` と表示されていればOKです。

### 静的なファイルで Hello World

`mkdir static` を実行し、静的なファイルを管理するディレクトリを作成します

```shell_session
mkdir static
```

その後、`static` ディレクトリ内に`index.html` を作成します。

```html:index.html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        Hello World!
    </body>
</html>
```

[`Iron`](https://github.com/iron/iron) 側で静的なファイルを使うためのライブラリを追加します。

 [`mount`](https://github.com/iron/mount) と [`staticfile`](https://github.com/iron/staticfile) です。

`Cargo.toml` に下記のように追加します。

```toml:Cargo.toml
[package]
name = "iron_vue"
version = "0.1.0"
authors = ["S-H-GAMELINKS <gamelinks007@gmail.com>"]
edition = "2018"

[dependencies]
iron = "0.6.0"
staticfile = "*"
mount = "*"
```

最後に、`src/main.rs` を下記のように変更し、`cargo run` を実行します。

```rust:src/main.rs
extern crate iron;
extern crate staticfile;
extern crate mount;

use iron::prelude::*;
use staticfile::Static;
use mount::Mount;
use std::path::Path;

fn main() {
    let mut mount = Mount::new();

    mount.mount("/", Static::new(Path::new("static/index.html")));

    Iron::new(mount).http("localhost:3000").unwrap();
}
```

```shell_session
cargo run
```

`localhost:3000` にアクセスし、`Hello World` と表示されていれば静的なファイルが使用できています。

### Vue.jsでHello World

[`Vue.js`](https://github.com/vuejs/vue) で `Hello World` とブラウザに表示させてみましょう。


`static` ディレクトリ内に `package.json` を作成します。

```json:package.json
{

}
```

`package.json` の中には `{}` だけで構いません。

次に、`yarn add vue` を実行します。

```shell_session
yarn add vue
```

コマンド実行後、 `package.json` が下記のように変更されていれば [`Vue.js`](https://github.com/vuejs/vue) はインストールされています。

```json:package.json
{
    "dependencies": {
        "vue": "^2.5.21"
    }
}
```

このままでは [`Vue.js`](https://github.com/vuejs/vue) が使えないので、 [`Webpack`](https://github.com/webpack/webpack) を導入します。

`static` ディレクトリ内に `webpack.config.js` を作成します。

```js:static/webpack.config.js
module.exports = {
    entry: './src/index.js', 
    output: { 
      filename: 'index.js',     
      path: `${__dirname}` 
    },
    resolve: {
        alias: {
          'vue$': 'vue/dist/vue.esm.js'
        }
    }
};
```

そして、 [`Webpack`](https://github.com/webpack/webpack) と [`webpack-cli`](https://github.com/webpack/webpack-cli) を [`yarn`](https://yarnpkg.com/lang/ja/)  でインストールします

```shell_session
yarn add webpack webpack-cli
```

上記のコマンドを実行すると、 `package.json` が以下のようになっていると思います。

```json:package.json
{
    "dependencies": {
        "vue": "^2.5.21",
        "webpack": "^4.28.4",
        "webpack-cli": "^3.2.1"
    }
}
```

`yarn` でビルドできるように `scripts` を追加します。

```json:package.json
{
    "scripts": {
        "build": "webpack --display-error-details"
    },
    "dependencies": {
        "vue": "^2.5.21",
        "webpack": "^4.28.4",
        "webpack-cli": "^3.2.1"
    }
}
```

`static/` ディレクトリ内に `src` ディレクトリを作成します

```shell_session
mkdir src
```

そして、`static/src` ディレクトリ内に `index.js` を作成します。

```js:static/src/index.js
import Vue from 'vue';

const app = new Vue({
    el: ".app",
    data: function() {
        return {
            text: "Hello World!"
        }
    }
})
```

これで `static` ディレクトリ内で `yarn build` を実行すると `index.js` がビルドされます。

`static/index.hml` を下記のように変更し、ビルドされた `index.js` を使用できるようにします。

```html:static/index.html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <div class="app">
            {{text}}
        </div>
        <script src="./index.js"></script>
    </body>
</html>
```

最後に、[`Iron`](https://github.com/iron/iron) 側で `index.js` を読み込めるようにします。

```rust:src/main.rs
extern crate iron;
extern crate staticfile;
extern crate mount;

use iron::prelude::*;
use staticfile::Static;
use mount::Mount;
use std::path::Path;

fn main() {
    let mut mount = Mount::new();

    mount.mount("/", Static::new(Path::new("static/index.html")));
    mount.mount("/index.js", Static::new(Path::new("static/index.js")));

    Iron::new(mount).http("localhost:3000").unwrap();
}
```

`cargo run` を実行し、ローカルサーバーを起動します。

```shell_session
cargo run
```

`localhost:3000` にアクセスし、`Hello World` と表示されていれば [`Vue.js`](https://github.com/vuejs/vue) が使用できています。

## SPAなWebサイトを作る
### Bootstrap Umiの導入

今のままではデザインが簡素すぎるので、 [`Bootstrap Umi`](https://github.com/ysakasin/umi) を 使います。

```shell_session
yarn add bootstrap-umi
```

次に、`static/src/index.js` を下記のように変更します。

```js:static/src/index.js

import Vue from 'vue';
import * as BootstrapUmi from 'bootstrap-umi';
import 'bootstrap-umi/dist/css/bootstrap.css';

Vue.use(BootstrapUmi);

const app = new Vue({
    el: ".app",
    data: function() {
        return {
            text: "Hello World!"
        }
    }
})
```

このまま `yarn build` したいところですが、`CSS` などを読み込む設定が [`Webpack`](https://github.com/webpack/webpack) 側で書かれていないのでビルドできません。

まず、[`style-loader`](https://github.com/webpack-contrib/style-loader) と [`css-loader`](https://github.com/webpack-contrib/css-loader) をインストールします

```shell_session
yarn add style-loader css-loader
```

`static/package.json` が以下のように変更されていれば、インストールされています。

```json:package.json
{
    "scripts": {
        "build": "webpack --display-error-details"
    },
    "dependencies": {
        "bootstrap-umi": "^4.0.0",
        "css-loader": "^2.1.0",
        "style-loader": "^0.23.1",
        "vue": "^2.5.21",
        "webpack": "^4.28.4",
        "webpack-cli": "^3.2.1"
    }
}
```

次に、`static/webpack.config.js` を変更し、ビルドできるようにします。

```js:static/webpack.config.js
module.exports = {
    entry: './src/index.js', 
    output: { 
      filename: 'index.js',     
      path: `${__dirname}` 
    },
    module: {
        rules: [
            {
                test: /\.css/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    },
    resolve: {
        alias: {
          'vue$': 'vue/dist/vue.esm.js'
        }
    }
};
```

その後、 `static` ディレクトリ内で `yarn vuild` を実行し、`index.js` をビルドします。

```shell_session
yarn build
```

確認のため `cargo run` を実行し、ローカルサーバーを起動します。

```shell_session
cargo run
```

`localhost:3000` にアクセスし、`Hello World` の字体が変更されていれば、[`Bootstrap Umi`](https://github.com/ysakasin/umi) が使用できています。


### Vueコンポーネントの導入

折角 [`Vue.js`](https://github.com/vuejs/vue) を使えるようにしているので `Vue コンポーネント` も使えるようにしたいですよね？

まずは、必要なライブラリを [`yarn`](https://yarnpkg.com/lang/ja/) でインストールします。
ちなみに追加するライブラリは [`vue-loader`](https://github.com/vuejs/vue-loader)、 [`vue-template-compiler`](https://www.npmjs.com/package/vue-template-compiler) です

```shell_session
yarn add vue-loader vue-template-compiler
```

`static/package.json` が下記のようになっていればOKです。

```json:static/package.json
{
    "scripts": {
        "build": "webpack --display-error-details"
    },
    "dependencies": {
        "bootstrap-umi": "^4.0.0",
        "css-loader": "^2.1.0",
        "style-loader": "^0.23.1",
        "vue": "^2.5.21",
        "vue-loader": "^15.5.1",
        "vue-template-compiler": "^2.5.21",
        "webpack": "^4.28.4",
        "webpack-cli": "^3.2.1"
    }
}
```

次に、 `Vue コンポーネント` を [`Webpack`](https://github.com/webpack/webpack) で使えるようにします。

```js:static/webpack.config.js
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
    entry: './src/index.js', 
    output: { 
      filename: 'index.js',     
      path: `${__dirname}` 
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                use: 'vue-loader'
            },
            {
                test: /\.css/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    },
    resolve: {
        alias: {
          'vue$': 'vue/dist/vue.esm.js'
        }
    },
    plugins: [
        new VueLoaderPlugin()
    ]
};
```

これで `Vue コンポーネント` が使用できるようになりました。

`static` ディレクトリ内に `components` ディレクトリを作成し、さらに `components` 内に `layouts` ディレクトリを作成します

```shell_session
mkdir components
cd components
mkdir layouts
```

`static/components/layouts` ディレクトリ内に `Header.vue` を作成します。

```vue:static/components/layouts/Header.vue
<template>
    <div>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
            <a class="navbar-brand" href="#">Iron Vue</a>
            <div class="dropdown">
                <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                    Menu
                </button>
                <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                    <a href="/" class="dropdown-item">Top</a>
                    <a href="/about" class="dropdown-item">About</a>
                    <a href="/contact" class="dropdown-item">Contact</a>
                </div>
            </div>
        </nav>
    </div>    
</template>
```

そして `static/src/index.js` で `Header.vue` を `import` します。

```js:static/src/index.js
import Vue from 'vue';
import * as BootstrapUmi from 'bootstrap-umi';
import 'bootstrap-umi/dist/css/bootstrap.css';

import Header from '../components/layouts/Header.vue';

Vue.use(BootstrapUmi);

const app = new Vue({
    el: ".app",
    components: {
        'nav-bar': Header
    },
    data: function() {
        return {
            text: "Hello Iron & Vue.js"
        }
    }
})
```

あとは、 `static/index.html` で `<nav-bar></nav-bar>` を追加します。

```html:static/index.html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <div class="app">
            <nav-bar></nav-bar>
            {{text}}
        </div>
        <script src="./index.js"></script>
    </body>
</html>
``` 

`static` ディレクトリ内で `yarn vuild` を実行し、`index.js` をビルドします。

```shell_session
yarn build
```

再び、確認のため `cargo run` を実行し、ローカルサーバーを起動します。

```shell_session
cargo run
```

`localhost:3000` にアクセスし、`Header.vue` の内容が表示されていればOKです


### vue-routerの導入

SPAなWebサイトにするので、[`vue-router`](https://github.com/vuejs/vue-router) をインストールします。

```shell_session
yarn add vue-router
```

`static/package.json` が以下のようになっていればインストールできています。

```json:static/package.json
{
    "scripts": {
        "build": "webpack --display-error-details"
    },
    "dependencies": {
        "bootstrap-umi": "^4.0.0",
        "css-loader": "^2.1.0",
        "style-loader": "^0.23.1",
        "vue": "^2.5.21",
        "vue-loader": "^15.5.1",
        "vue-router": "^3.0.2",
        "vue-template-compiler": "^2.5.21",
        "webpack": "^4.28.4",
        "webpack-cli": "^3.2.1"
    }
}
```

次に、表示する各ページのコンポーネントを作成します。

`static/components` ディレクトリ内に `webs` ディレクトリを作成します。

```shell_session
mkdir webs
```

`static/components/webs` ディレクトリ内に `Index.vue`、`About.vue`、`Contact.vue` を追加します。

```vue:static/components/webs/Index.vue
<template>
    <div class="container">
        <h1>Index Pages</h1>
    </div>
</template>
```

```vue:static/components/webs/About.vue
<template>
    <div class="container">
        <h1>About Pages</h1>
    </div>
</template>
```

```vue:static/components/webs/Contact.vue
<template>
    <div class="container">
        <h1>Contact Pages</h1>
    </div>
</template>
```

`static` ディレクトリ内に `router` ディレクトリを作成します。

```shell_session
mkdir router
```

`static/router` ディレクトリ内に、`router.js` を作成します。

```js:static/router/router.js
import Vue from 'vue';
import VueRouter from 'vue-router';
import Index from '../components/webs/Index.vue';
import About from '../components/webs/About.vue';
import Contact from '../components/webs/Contact.vue';

Vue.use(VueRouter)

export default new VueRouter({
  mode: 'history',
  routes: [
    { path: '/', component: Index },
    { path: '/about', component: About },
    { path: '/contact', component: Contact },
  ],
})
```

次に、`static/src/index.js` に `static/router/router.js` を `import` します。

```js:static/src/index.js
import Vue from 'vue';
import * as BootstrapUmi from 'bootstrap-umi';
import 'bootstrap-umi/dist/css/bootstrap.css';

import Header from '../components/layouts/Header.vue';

import Router from '../router/router';

Vue.use(BootstrapUmi);

const app = new Vue({
    el: ".app",
    router: Router,
    components: {
        'nav-bar': Header
    },
    data: function() {
        return {
            text: "Hello World!"
        }
    }
})
```

そして、`static/components/layouts/Header.vue` と `static/index.html` を下記のように修正します。

```vue:static/components/layouts/Header.vue
<template>
    <div>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
            <router-link to="/" class="navbar-brand">Iron Vue</router-link>
            <div class="dropdown">
                <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                    Menu
                </button>
                <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                    <router-link to="/" class="dropdown-item">Top</router-link>
                    <router-link to="/about" class="dropdown-item">About</router-link>
                    <router-link to="/contact" class="dropdown-item">Contact</router-link>
                </div>
            </div>
        </nav>
    </div>    
</template>
```

```html:static/index.html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>
    <body>
        <div class="app">
            <nav-bar></nav-bar>
            <div class="constainer">
                <router-view></router-view>
            </div>
            {{text}}
        </div>
        <script src="./index.js"></script>
    </body>
</html>
```

ただ、このままではリロードした際に各ページが表示されません。

そこで、[`Iron`](https://github.com/iron/iron) へルーティングを追加します。

```rust:src/main.rs
extern crate iron;
extern crate staticfile;
extern crate mount;

use iron::prelude::*;
use staticfile::Static;
use mount::Mount;
use std::path::Path;

fn main() {
    let mut mount = Mount::new();

    let routes = ["/", "/about", "/contact"];

    for route in &routes {
        mount.mount(route, Static::new(Path::new("static/index.html")));
    }

    mount.mount("/index.js", Static::new(Path::new("static/index.js")));

    Iron::new(mount).http("localhost:3000").unwrap();
}
```

最後に、確認のため `cargo run` を実行し、ローカルサーバーを起動します。

```shell_session
cargo run
```

`localhost:3000` にアクセスし、`Menu` のリンクをクリックしてページが切り替わればOKです。

これで、SPAなWebサイトは完成です！

## Herokuへデプロイ 

いよいよ、[`Heoku`](https://jp.heroku.com/) へとデプロイしたいと思います。

Herokuへのデプロイボタンを使用してデプロイします。

まず`README.md` を作成し、[`Heoku`](https://jp.heroku.com/) へのデプロイボタンを追加します。

```markdown:README.md
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)
```

次に、`app.json` を追加します。

```json:app.json
{
    "name": "Iron Vue",
    "description": "SPA Web Application Sample for Iron & Vue.js ",
    "website": "https://github.com/S-H-GAMELINKS/iron_vue",
    "repository": "https://github.com/S-H-GAMELINKS/iron_vue",
    "buildpacks": [
      {
        "url": "https://github.com/emk/heroku-buildpack-rust.git"
      }
    ],
    "logo": "https://small-sharp-tool.com/logo.svg",
    "success_url": "/"
}
```

そして、`Procfile` を追加します。

```Procfile:Procfile
web: ./target/release/iron_vue
```

[`Heoku`](https://jp.heroku.com/) では `port` を自動的に割り当ています。
そのため、現状のコードではデプロイはできてもWebサイトが表示されないことになります。

そこで、`src/main.rs` を以下のように変更します。

```rust:src/main.rs
extern crate iron;
extern crate staticfile;
extern crate mount;

use iron::prelude::*;
use staticfile::Static;
use mount::Mount;
use std::path::Path;
use std::env;

fn get_server_port() -> u16 {
    env::var("PORT").ok()
        .and_then(|p| p.parse().ok())
        .unwrap_or(3000)
}

fn main() {
    let mut mount = Mount::new();

    let routes = ["/", "/about", "/contact"];

    for route in &routes {
        mount.mount(route, Static::new(Path::new("static/index.html")));
    }

    mount.mount("/index.js", Static::new(Path::new("static/index.js")));

    Iron::new(mount).http(("0.0.0.0", get_server_port())).unwrap();
}
```

これで、自動的に割り当てられた `port` を取得することができます。

これまでの変更をコミットし、`GitHub` へ `push` します。

```shell_session
git init
git add .
git commit -am "deploy to Heroku"
git push origin master
```

あとは`Deploy To Heroku` ボタンを押すだけです。


# 参考

[Deploying Rust applications to Heroku, with example code for Iron](http://www.randomhacks.net/2014/09/17/deploying-rust-heroku-iron/)

[C++/Vue.js/WebpackでSPAサンプルを作った話](https://qiita.com/S_H_/items/620ee3f2765e1446a851)

---
title: "Webpack入門チュートリアル Vue.js環境構築編"
date: 2018-10-18T18:03:03+09:00
desciption: aaaaaaa
---

本記事ではWebpackを用いて、Vue.jsの環境構築を行います。
vue-cliを使わずに1からつくっていくことで、webpack初学者の方がwebpackの基礎を理解できるようにしています。

<!--more--> 


# 目次
1. webpackの導入
2. webpack.config.jsの設定
3. ES6のトランスパイル(babel導入)
4. .vue拡張子への対応
5. scssへの対応
6. webpack-dev-serverの導入


## 1. webpackの導入

さっそくwebpackの導入を行いましょう。

### 1-1. packageのインストール
まずはwebpackの実行に必要なパッケージをnpmでインストールしていきましょう。

```
npm init -y
npm install -D webpack webpack-cli
```

### 1-2. ファイル作成

次に、JavaScriptのファイルを作成します。  
今回はsrc/index.jsというファイルを作成しましょう。  

```
mkdir -p src
touch src/index.js
```

つづいてsrc/index.jsを編集していきましょう。

``` src/index.js
console.log('Hello World');
```

## 2. webpack.config.jsの作成
プロジェクトルートディレクトリ直下に webpack.config.js というファイルを作成します。これはwebpackの設定を記述するためのファイルです。

このファイルを以下のように編集します。

```
const path = require('path');

module.exports = {
  entry: {
    index: './src/index.js'
  },
  output: {
    filename: '[name].js',
    path: path.join(__dirname, 'dist')
  }
}

```

### entry
webpackのビルド処理を開始する起点となるファイル、すなわちエントリーポイントをを指定します。
webpackはエントリーポイントに指定したファイル内のimportやrequireなどの単語をみて、それ以外のすべてのファイルとの依存関係をいい感じに管理してくれます。


### output
webpackは複数のJavaScriptファイルのの依存関係を整理し1つのJavaScriptファイルにbundleします。そのbundleしたファイルの出力先を指定するのがoutputです。

また、ここで[name]を使用すると、entryで指定したkeyがファイル名になります。
今回のケースでは webpackを実行すると、dist/index.js というファイルが生成されます。


### webpackを実行してみましょう。
以下のコマンドでwebpackを実行しましょう。

```
./node_modules/.bin/webpack
```

ビルドが成功したら dist/index.js が生成されているはずです。
nodeで実行して結果を見てみましょう。

```
node dist/index.js
=> "Hello World"
```

## 3. LoaderをつかってES6をトランスパイルしよう
### 3-1. Loaderとは
webpackが理解できるのはJavaScriptとJSONのみです。しかし、Loaderを使用することで異なる拡張子・言語のファイルをJavaScriptに変換してweboackで管理できるようにすることができます。

(例)
``` webpack.config.js
// npm install -D raw-loader

const path = require('path');

module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
      // .txtファイルをwebpackで管理できるようになります。
    ]
  }
};

```

### 3-2. babel-loaderでES6をトランスパイル
[babel-loader](https://github.com/babel/babel-loader)を用いてES6をトランスパイルしましょう。
まずはnpmで必要なパッケージをインストールしましょう。(参照: [Github:babel-loader](https://github.com/babel/babel-loader))

```
npm install -D babel-loader @babel/core @babel/preset-env
```

つづいてwebpack.config.jsにloaderの設定を追記します。

```
const path = require('path');

module.exports = {
  entry: {
    index: './src/index.js'
  },
  output: {
    filename: '[name].js',
    path: path.join(__dirname, 'dist')
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  }
}

```

Loader設定の基本は、rulesというオブジェクトの中に以下を記述していくことです。
	- どの拡張子のファイルに対して
	- どのloaderを使うか

各項目の説明をします。

#### test
「どの拡張子に対して」の部分を担います。

#### exclude
loader適用の対象外とするディレクトリを指定します。

#### use
testで指定した拡張子に対して「どのloaderを使うか」を記述します。
別途、optionをつけて挙動を細かく変えることができます。


### 3-3. トランスパイル結果を確認してみましょう。

src/index.jsを以下のようにES6のクラス構文を使ったものに書き換えてください。

``` src/index.js
class Person {
  constructor(name) {
    this.name = name
  }

  hello() {
    console.log(`Hello. I am ${this.name}`)
  }
}

person = Person.new('Tommy');
person.hello();
```

webpackを実行しましょう。

```
./node_modules/.bin/webpack
```

bundleされたファイル、すなわちdist/index.jsの中身を確認しましょう。
class Person という記述が見つからず、function Person() に変換されているのが確認できましたでしょうか。
これでES6をES5へとトランスパイルすることに成功しました。

## 4. .vue拡張子への対応
### 4-1. パッケージのインストール
Vue.jsの利用に必要なパッケージを入れていきましょう。

```
npm install -S vue
npm install -D vue-loader vue-template-compiler
```

|パッケージ|説明|
|---:|---:|
|vue|Vue.js本体|
|vue-loader|.vue拡張子を読み込むためのloader|
|vue-template-compiler|template構文の利用に必要なパッケージ|

### 4-2. Loaderの定義

webpack.config.jsにて、Loaderを以下のように定義しましょう。

``` webpack.config.js
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
  entry: {
    index: './src/index.js'
  },
  output: {
    filename: '[name].js',
    path: path.join(__dirname, 'dist')
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },

      {
        test: /\.vue$/,
        exclude: /node_modules/,
        use: {
          loader: 'vue-loader',
        }
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ]
}
```



### 4-3. Vue.jsのSFCを作成しよう

src配下にApp.vueを作成しましょう。


``` src/App.vue

<template>
  <div>
    <h1>HelloWorld</h1>
    <p>Your name {{ name }}</p>
    <input v-model="name">
  </div>
</template>

<script>

export default {
  data() {
    return {
      name: 'Tommy'
    }
  }
}

</script>

```


src/index.jsでApp.jsをimportし、htmlにmountする記述を書きます。

``` src/index.js
import Vue from 'vue';
import App from './App.vue';

new Vue(App).$mount('#hello')
```

最後にindex.htmlをdist配下に作成しましょう。

``` dist/index.html

<!DOCTYPE html>
<html>
<head>
  <title></title>
</head>
<body>


<div id="hello"></div>

<script src="index.js"></script>
</body>
</html>


```


### 4-4. webpackの実行
webpackを実行。

```
./node_modules/.bin/webpack
```

index.htmlをブラウザで開いてみましょう！
以下のような簡単なVue.jsができているはずです！

[![Image from Gyazo](https://i.gyazo.com/b9792f87b0d5d4249bf89a4538e0a83a.gif)](https://gyazo.com/b9792f87b0d5d4249bf89a4538e0a83a)


## 5. scssへの対応
つづいてSass/Scssを利用できるようにしてみましょう。


### 5-1. Loaderのインストール
以下のパッケージをインストールします。

```
npm install -D vue-style-loader css-loader sass-loader node-sass
```

|パッケージ|説明|Github|
|---:|---:|---:|
|vue-style-loader|CSSをstyleタグに変換してbundle.jsに出力してくれます。|https://github.com/vuejs/vue-style-loader|
|css-loader|CSSをJavaScriptに変換してくれるLoaderです。|https://github.com/webpack-contrib/css-loader|
|sass-loader|SassをCSSにコンパイルしてくれうローダーです。|https://github.com/webpack-contrib/sass-loader|
|node-sass|SassをコンパイルするためのNode.js用ライブラリです。sass-loaderがこいつを動かします|https://github.com/sass/node-sass|

#### 参考記事
https://qiita.com/shuntksh/items/bb5cbea40a343e2e791a


### 5-2 Loaderの設定

webpack.config.jsを以下のように設定して、.css, .scss拡張子のファイルに対してLoaderを用いるように設定しましょう。

```

const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
  entry: {
    index: './src/index.js'
  },
  output: {
    filename: '[name].js',
    path: path.join(__dirname, 'dist')
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },

      {
        test: /\.vue$/,
        exclude: /node_modules/,
        use: {
          loader: 'vue-loader',
        }
      },

      {
        test: /\.(scss|css)$/,
        exclude: /node_modules/,
        use: [
          'vue-style-loader',
          'css-loader',
          'sass-loader',
        ]
      },
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ]
}


```


### 5-3 App.vueにscssを記述してみましょう。

以下のようにApp.vueにscssを記述してみましょう。

``` App.vue

<template>
  <div>
    <h1>HelloWorld</h1>
    <p>Your name {{ name }}</p>
    <input v-model="name">
  </div>
</template>

<script>

export default {
  data() {
    return {
      name: 'Tommy'
    }
  }
}

</script>


<style scope lang="scss">

div {
  h1 {
    color: red;
  }
}

</style>


```


webpackにトランスパイルをさせて表示しましょう。

```
./node_modules/.bin/webpack
```

ブラウザでindex.htmlを開いて以下のようになっていればOKです。

[![Image from Gyazo](https://i.gyazo.com/85bd24daf373a0e8758fd7d27ba71ab1.png)](https://gyazo.com/85bd24daf373a0e8758fd7d27ba71ab1)



## 6. webpack-dev-serverへの対応
毎回毎回./node_modules/.bin/webpackを呼び出してトランスパイルをするのは面倒なので、webpackは開発用のサーバーを用意してくれています。それがwebpack-dev-serverです。ファイルの変更を検知して自動的にトランスパイルをしなおしてくれるhot-reload機能を持っているため非常に便利です。


### 6-1. パッケージのインストール

```
npm install -D webpack-dev-server
```


### 6-2. webpack.config.jsの設定
以下の設定を記述しましょう。


``` webpack.config.js

// 省略

module.exports =  {
  // 省略 

  module: {
    // 省略
  },
  plugins: [
    // 省略
  ],

  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    port: 3000,
    index: 'index.html'
  }
}

```


|項目|詳細|
|---:|---:|
|contentBase|ここで指定したディレクトリ直下のファイルをserveします|
|port|開発サーバーのport番号を指定します。|
|index|indexファイルを指定します。これを指定するとルートパスでこのファイルをserveします。|


### 6-3. webpack-dev-serverを起動しよう。

package.jsonのscript内に以下を記述しましょう。

```
"scripts": {
  "start": "webpack-dev-server --hot",
}
```

```
npm run start
```

で起動です。



[![Image from Gyazo](https://i.gyazo.com/2658af54ce5f3eb1249d4ded9459f14e.gif)](https://gyazo.com/2658af54ce5f3eb1249d4ded9459f14e)


これで本記事は終了です！

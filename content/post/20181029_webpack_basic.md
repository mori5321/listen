---
title: "Webpack入門チュートリアル Vue.js環境構築編"
date: 2018-10-18T18:03:03+09:00
---

本記事ではWebpackを用いて、Vue.jsの環境構築を行います。
vue-cliを使わずに1からつくっていくことで、webpack初学者の方がwebpackの基礎を理解できるようにしています。


## 目次
1. webpackの導入
2. webpack.config.jsの設定
3. ES6のトランスパイル(babel導入)
4. .vue拡張子への対応
5. scssへの対応
6. webpack-dev-serverの導入
7. 絶対パス指定でのファイルimport



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

つづいてsrc/javascript/index.jsを編集していきましょう。

``` src/javascript/index.js
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
    path: path.join(__dirname, 'dist/javascript')
  }
}

```

### entry
webpackのビルド処理を開始する起点となるファイル、すなわちエントリーポイントをを指定します。
webpackはエントリーポイントに指定したファイル内のimportやrequireなどの単語をみて、それ以外のすべてのファイルとの依存関係をいい感じに管理してくれます。


### output
webpackは複数のJavaScriptファイルのの依存関係を整理し1つのJavaScriptファイルにbundleします。そのbundleしたファイルの出力先を指定するのがoutputです。

また、ここで[name]を使用すると、entryで指定したkeyがファイル名になります。
今回のケースでは webpackを実行すると、dist/javascript/index.js というファイルが生成されます。


### webpackを実行してみましょう。
以下のコマンドでwebpackを実行しましょう。

```
./node_modules/.bin/webpack
```

ビルドが成功したら dist/javascript/index.js が生成されているはずです。
nodeで実行して結果を見てみましょう。

```
node dist/javascript/index.js
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
    path: path.join(__dirname, 'dist/javascript')
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

bundleされたファイル、すなわちdist/javascript/index.jsの中身を確認しましょう。
class Person という記述が見つからず、function Person() に変換されているのが確認できましたでしょうか。
これでES6をES5へとトランスパイルすることに成功しました。

## 4. .vue拡張子への対応
Vue.jsの利用に必要なパッケージを入れていきましょう。

```
npm install -S vue
npm install -D vue-loader vue-template-compiler
```

|パッケージ名|説明|
|---:|---:|
|vue|Vue.js本体|
|vue-loader|.vue拡張子を読み込むためのloader|
|vue-template-compiler|template構文の利用に必要なパッケージ|


## 5. scssへの対応

## 6. webpack-dev-serverへの対応

## 7. 絶対パス指定でのファイルimportができるようにする




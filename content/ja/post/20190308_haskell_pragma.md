---
title: "20190308_haskell_pragma"
date: 2019-03-08T08:25:08+09:00
draft: false
---


私のにわかプログラマーがHaskellに手をだすと出くわすこいつ。

{-# #-}

``` samp.hs
{-# LANGUAGE OverloadedStrings #-}
```

APIなんてたてようものなら
Compilerの警告に従うままに"こいつら"を追加するといつのまにかこんなになってる。


```
{-# LANGUAGE DataKinds                  #-}
{-# LANGUAGE DeriveGeneric              #-}
{-# LANGUAGE FlexibleInstances          #-}
{-# LANGUAGE GADTs                      #-}
{-# LANGUAGE GeneralizedNewtypeDeriving #-}
{-# LANGUAGE MultiParamTypeClasses      #-}
{-# LANGUAGE OverloadedStrings          #-}
{-# LANGUAGE QuasiQuotes                #-}
{-# LANGUAGE TemplateHaskell            #-}
{-# LANGUAGE TypeFamilies               #-}
{-# LANGUAGE TypeOperators              #-}
```


得体の知れない記法。
参考書ではなぜかほとんど言及されない。
彼らはいったい何者なのか。

<!--more-->


# どうやらPragmaというらしい

[公式wiki](https://wiki.haskell.org/Language_Pragmas) を見た所、Pragmaと呼ぶらしいということがわかった。

重要なのは以下の2点。

> A language pragma directs the Haskell compiler to enable an extension or modification of the Haskell language. 

> Language pragmas in a Haskell module must appear at the very beginning of the module.


- "Language pragma"はHaskellコンパイラーに対して、Haskellの言語拡張や修正をコンパイルできるように指示をします。
- "Language pragma"はファイルの一番上に書かねばなりません。






# 頻出Language Pragma5種
自分が初めてAPIを建てたときに、よく出てくるLanguage Pragma5つを調べてみました。

## OverloadedStrings
Haskellのnumericリテラルはデフォルトで以下のようにポリモーフィックである。IntにもDoubeにもFloatにもなれる。

```
a :: Int
a = 1

b :: Double
b = 1

c :: Float
c = 3.5

d :: Rational
d = 3.5
```

一方でStringリテラル("")は、常にString型である。つまりポリモーフィックではない。*OverloadedStrings*は、Stringリテラルを「IsString型クラス」に対してポリモーフィックにしてくれる。これによってText型やBytestring型にもSrtringリテラルを使えるようになる。

```
import Data.Text

a :: String
a = "hello"

b :: Text
b = "Hello"
# => Compile Error

```

```
{-# LANGUAGE OverloadedStrings #-}
import Data.Text

a :: String
a = "hello"

b :: Text
b = "Hello"
# => Compiled!
```

参考: https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/guide-to-ghc-extensions/basic-syntax-extensions#overloadedstrings

## DataKinds

## TemplateHaskell

## TypeOperators

## DeriveGeneric


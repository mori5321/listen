---
title: "hanamiにおけるassosiation"
date: 2018-11-16T21:30:00+09:00
description: Webpack for Beginner with Vue.js
---

*[公式ドキュメント](https://guides.hanamirb.org/associations/has-many/)* を見ると、ある1つの親モデルから複数の子モデルを取得するコードサンプルが載っている。


``` samp.rb

repo = QuestionGroupRepository.new
repo.aggregate(:books).where(id: id).map_to(Author).one

```

しかし、複数の親モデルから子モデルを取得する方法は載っておらず少々苦戦した。
(依存ライブラリであるROMのドキュメントを見た。結論以下の記述で出すことができる)

<!--more-->


``` samp.rb
repo = QuestionGroupRepository.new
repo.aggregate(:questions).map_to(QuestionGroup).to_a
```

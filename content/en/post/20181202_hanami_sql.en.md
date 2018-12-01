---
title: "A Strong Ideology of Hanami about SQL separation"
date: 2018-12-01T21:30:00+09:00
---


I felt a strong ideology of Hanami about SQL separation.

Hanami strolongy retrict the use of where method on Repotisory.
"Where" cannot be called as public method, only as private method.

``` samp.rb
@repo = UserRepository.new

# NG: @repo.where(name: "moris")
```

<!--more-->

So, we've got to define method by ourselves.


``` samp.rb

class UserRepository < Hanami::Repository
  def find_by_name(name)
    users.where(name: name).first
  end
end

```


This ideology is totally diffrerent from the one of Ruby on Rails.
Rails permit us to call where method whereever you like.


``` samp.rb
class UsersController < ApplicationController
  def index
    @user = User.where(status: params[:status], banned: false)
  end
end

```

However, this leaves a strong dependency from Controller to SQL internal implementation, such as, "whether UserRepositry has banned?".

Hanami is more Single-Reponsibility-Like framework.
It prefer to separte SQL from Client-Side Class

In hanami documentation, the reason it restrict 'public where' is explained.
- Caller depends on Repository internal structrure
- If Caller think about SQL, abstraction levels of layers get not aligned.
- Method chain does not explain its intention.
- Less Testable


I love this thought of Complete Separation of Client and DomainModel.
Even if you are bored with define such a fetching method on repository, you can use meta-programming.



---
title: "GraphQL Ruby Sample"
date: 2018-12-05T21:30:00+09:00
---

To instantly understand GraphQL concept, I made a sample code of by Ruby.

<!--more-->


```
require 'graphql'
require 'active_support'

# Repository
module PersonRepository
  extend ActiveSupport::Concern
  @@persons = []

  included do
    def self.fetch(limit: )
      limit = @@persons.length unless limit
      @@persons[0..limit]
    end

    def self.find_by_name(name)
      @@persons.select{|p| p.name == name }
    end
  end

  def add_person(person)
    @@persons << person
  end
end

# define Person Class
class Person
  include PersonRepository

  def initialize(id: rand(100), name: 'Tom')
    @id = id
    @name = name
    add_person(self)
  end

  attr_reader :id, :name
end


# generate person Object
names = ['Tom', 'Jelly', 'Moris', 'gitgit', 'Gitlab', "Halohalo"]
persons = names.map {|n| Person.new(name: n) }


# define Person Schema
PersonType = GraphQL::ObjectType.define do
  name 'Person'

  field :id, types.ID
  field :name, types.String
end


# define Query Schema
QueryType = GraphQL::ObjectType.define do
  name "Query"

  field :persons, types[PersonType] do
    argument :limit, types.Int
    resolve -> (obj, args, ctx) {
      Person.fetch(limit: args[:limit])
    }
  end

  field :persons_by_name, types[PersonType] do
    argument :name, !types.String, '名前で検索'
    resolve -> (obj, args, ctx) { Person.find_by_name(args[:name]) }
  end
end


# Schema
PersonSchema = GraphQL::Schema.define do
  query QueryType
end


puts "----"

# ExecuteQuery
p result = PersonSchema.execute('
  {
    persons {
      id name
    }
  }
').to_h

puts "----"

# ExecuteQuery
p result = PersonSchema.execute('
  {
    persons(limit: 3) {
      id name
    }
  }
').to_h

puts "----"

p result2 = PersonSchema.execute('
  {
    persons_by_name(name: Tom) {
      id name
    }
  }
').to_h

```


Results are like this.

``` .bash

----
{"data"=>{"persons"=>[{"id"=>"18", "name"=>"Tom"}, {"id"=>"71", "name"=>"Jelly"}, {"id"=>"4", "name"=>"Moris"}, {"id"=>"79", "name"=>"gitgit"}, {"id"=>"48", "name"=>"Gitlab"}, {"id"=>"74", "name"=>"Halohalo"}]}}
----
{"data"=>{"persons"=>[{"id"=>"18", "name"=>"Tom"}, {"id"=>"71", "name"=>"Jelly"}, {"id"=>"4", "name"=>"Moris"}, {"id"=>"79", "name"=>"gitgit"}]}}
----
{"data"=>{"persons_by_name"=>[{"id"=>"18", "name"=>"Tom"}]}}


```

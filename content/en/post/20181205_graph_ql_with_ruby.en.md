# GraphQL Ruby Sample

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
    def self.all
      @@persons
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
names = ['Tom', 'Jelly', 'Moris', 'gitgit']
persons = names.map {|n| Person.new(name: n) }


# define Person Schema
PersonType = GraphQL::ObjectType.define do
  name 'Person'

  field :id do
    type !types.ID
    resolve -> (obj, args, ctx) { rand(10) }
  end

  field :name do
    type !types.String
    resolve -> (obj, args, ctx) { 'Tom' }
  end
end


# define Query Schema
QueryType = GraphQL::ObjectType.define do
  name "Query"

  field :persons, types[PersonType] do
    resolve -> (obj, args, ctx) { Person.all }
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


# ExecuteQuery
p result = PersonSchema.execute('
  {
    persons {
      id name
    }
  }
').to_h

p result2 = PersonSchema.execute('
  {
    persons_by_name(name: Tom) {
      id name
    }
  }
').to_h
```

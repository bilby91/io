---
layout: post
title: "Rails to Hanami"
description: "Simple Hanami JSON API from someone with a Rails background"
tags: [hanami, rails]
published: true
---

A couple of month ago I discovered a small community of ruby developers that where building a new web
framework, [Hanami](http://hanamirb.org). Since then, I've been following pretty closely the progress of
the project. In the last version, Hanami integrated with [ROM](http://rom-rb.org) in their data layer, so,
I decided it was a good idea to play a little bit with it and create a simple REST API to see how it worked out.

# What we are going to build ?

We are going to build a REST JSON API for a classic TODO application

- [JSON API Standard](http://jsonapi.org/) for encoding of json.
- [Postgresql](https://www.postgresql.org) for data storage.

## Endpoints

### Lists

- GET - Fetch list of lists (/lists)
- GET - Fetch list (/lists/:id)
- POST - Create list (/lists)

### Tasks

- GET - Fetch tasks (/lists/:list_id/tasks)
- GET - Fetch task  (/lists/:list_id/tasks/:id)
- POST - Create task (/lists/:list_id/tasks)

## Entities

## List

- id (Integer)
- name (String)

## Task

- id (Integer)
- description (String)
- list_id (Integer)

# Development

Lets start by creating the project. Hanami comes with an initialization command just like rails.

```shell
hanami new hanami_todo_app --database=postgres
cd hanami_todo_app
bundle install
```

After running the new command, Hanami should have created a new directory with the name `hanami_todo_app`. We can `cd` into the directory and take a look at the structure. The [architecture](http://hanamirb.org/guides/architecture/overview/) of Hanami applications it's a little bit different than rails. The application is split in two main directories, lib and apps. In the lib directory most of your business logic will sit. This includes your entities (almost like a Rails model but without the persistence apis, more on this later), services, interactors or whatever code organization design you use for your application. Under the apps folder you will likely have only one folder in a simple app. We are going to have only one folder called `api` that is were all our http actions will sit. If you have a big application you will likely have multiple folders under apps, for example, admin (administrative UI), web, etc. Rails has engines for splitting big application into different modular smaller apps.

By default Hanami will create a default app called `web`, we are going to delete that application and create a new one called `api` just to help my OCD.

```shell
hanami destroy app web
hanami generate app api
```

After those command are executed we should have removed the `web` folder under `apps` and a new `api` folder should have been created.

## Entities and Repositories (Models)

We are going to start modeling our system now. Two simple entities should be enough, List and Task.

```shell
➜  hanami_todo_app git:(master) ✗ hanami generate model list
      create  lib/hanami_todo_app/entities/list.rb
      create  lib/hanami_todo_app/repositories/list_repository.rb
      create  db/migrations/20170531214217_create_lists.rb
      create  spec/hanami_todo_app/entities/list_spec.rb
      create  spec/hanami_todo_app/repositories/list_repository_spec.rb

➜  hanami_todo_app git:(master) ✗ hanami generate model task
      create  lib/hanami_todo_app/entities/task.rb
      create  lib/hanami_todo_app/repositories/task_repository.rb
      create  db/migrations/20170531214247_create_tasks.rb
      create  spec/hanami_todo_app/entities/task_spec.rb
      create  spec/hanami_todo_app/repositories/task_repository_spec.rb
```

As you can see from the command's output, Hanami created the entity but it also created a repository. We will discuss more on them later. Now we can go to the generated migration files and adapt them as we need.

```ruby
Hanami::Model.migration do
  change do
    create_table :lists do
      primary_key :id
      column :name, String, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

```ruby
Hanami::Model.migration do
  change do
    create_table :tasks do
      primary_key :id
      column :description, String, null: false
      foreign_key :list_id, :lists, on_delete: :cascade, null: false

      column :created_at, DateTime, null: false
      column :updated_at, DateTime, null: false
    end
  end
end
```

Hanami's migration DSL is very similar to the one ActiveRecord provides. We need to provide column name, data type and any constraint that we want for the column. In this cases we added nullability constraint to enforce non null columns. Hanami automatically generates accessors and setters for our entities based on the database schema, same as rails.

For running the migrations we can use `hanami db migrate`, since we didn't create the database yet we can run `hanami db prepare` that will create and run migrations for us. For more database related tasks you can check them with `hanami db`.

<!-- Let's try to create a new List using the Hanami console.  -->

One important difference between Hanami and Rails regarding data modeling is that Hanami doesn't provide an enormous api on the entities classes. Entities are very simple ruby classes that inherit from `Hanami::Entity`. They don't have api's for querying, creating or updating database records, the responsibility for doing all those tasks is encapsulated in the entitiy associated Repository (remember we mentioned them ?).

This change will be of high impact on the design of your system. By splitting the persistence from the model you are not coupling your business logic with the underlying data store. Every Repository has it's own independent data store, in our case the ListRepository and TaskRepository will be persisted in the postgres db.

Lets imagine that our application it's getting a lot of traffic and we detected that our SQL db is the bottleneck. One possible solution to the problem could be to move the highly accessed data to a faster data store, for example, redis. Instead of refactoring all our code we can simple change our Repository to start using a different data store and we won't need to actually touch our entity class. This is a super simple and naive example of how this can be useful by I hope you get the idea.

We are going to play with the `hanami console` and create some objects using the repository.

```ruby
# Let create a new repository instance
2.3.1 :011 > list_repository = ListRepository.new
 => #<ListRepository relations=[:lists]>

# Create and store a new list object
2.3.1 :012 > list_repository.create(name: 'Grocery')
[hanami_todo_app] [INFO] [2017-05-31 19:08:39 -0300] (0.000608s) SELECT "id", "name", "created_at", "updated_at" FROM "lists" LIMIT 1
[hanami_todo_app] [INFO] [2017-05-31 19:08:39 -0300] (0.003811s) INSERT INTO "lists" ("name", "created_at", "updated_at") VALUES ('Grocery', '2017-05-31 22:08:39.068319+0000', '2017-05-31 22:08:39.068319+0000') RETURNING *
 => #<List:0x007fa244403380 @attributes={:id=>1, :name=>"Grocery", :created_at=>2017-05-31 22:08:39 UTC, :updated_at=>2017-05-31 22:08:39 UTC}>
```

[Repositories](http://hanamirb.org/guides/models/repositories/) provide the apis for interacting with the persistence layer. From the Hanami docs:

> An object that mediates between entities and the persistence layer. It offers a standardized API to query and execute commands on a database.

The methods you are used to call in your Rails models are now in the repository, so, instead of doing `User.create({ ... })` you will do `UserRepository.new.create({ ... })`. Same applies for `find` and update, I suggest looking at the docs for a better understanding since apis are not exactly the same.

Repositories also encapsulate your custom queries. In Rails your probably created some scopes in the model to encapsulate your query logic. In Hanami you write methods in your repository. One big difference between Hanami and Rails in this case is that Hanami doesn't expose a method to perform custom queries outside the Repository class. This is a really good idea because you will be forced to write all your query logic in one place instead of having your queries spread all along your codebase :).

This is how our ListRepository looks like.

```ruby
class ListRepository < Hanami::Repository
  associations do
    has_many :tasks
  end

  def with_name(name)
    lists.where(name: name).as(List).to_a
  end

  def find_with_tasks(id)
    aggregate(:tasks).where(id: id).as(List).one
  end
end
```

As you can see, we have `with_name` that hides the logic of querying our datastore for a List with a given name.

The other important element that we define in the repository are the associations. This is the same concept that we have in `ActiveRecord`, it defines how our different entities/models relate to each other. This association definitions will be helpful when we want to query data based on multiple tables. At the moment Hanami only supports `has_many` associations but you can use the develop branch to get `belongs_to` associations too. Associations are very powerful, they are powered by `ROM`. At the moment the feature is experimental but I guess this is going to become stable as soon as more people start using Hanami.

## Actions (Rails controller actions)

We can move to our final step to get our api working, the actions (same as controller's methods in Rails).

Rails uses controllers to encapsulate the http requests handlers that your application has. Hanami decided to have one class per action instead of having a controller that groups the actions of a given resource.

Lets use the generators to create the action classes. We can do `hanami generate action api 'lists#index'` to create the index action.

Generator will add the correct routes in your config/routes file so you don't have to worry about that.

```ruby
module Api::Controllers::Lists
  class Index
    include Api::Action
    include JSONAPI::Hanami::Action

    def call(params)
      self.data = list_repository.all
      self.status = 200
    end

    def list_repository
      @list_repository ||= ListRepository.new
    end
  end
end
```

The important method here is `call`. This method will be called when the route that is associated with this action is requested. By using the gem `gem 'jsonapi-hanami'` (don't forget to add it in your Gemfile) we can serialize our Lists into a valid jsonapi response. For more information on this nice gem take a look at [here](http://jsonapi-rb.org/guides/getting_started/hanami.html).

Let's look at a different action now, one that receives data from the client.

```ruby
module Api::Controllers::Lists
  class Create
    include Api::Action
    include JSONAPI::Hanami::Action

    deserializable_resource :list

    params do
      required(:list).schema do
        required(:name)
      end
    end

    def call(params)
      self.data = list_repository.create(params[:list])
      self.status = 200
    end

    def list_repository
      @list_repository ||= ListRepository.new
    end
  end
end
```

In this actions we have some other elements to discuss. First, `deserializable_resource` is telling the parser to parse the incoming payload looking for a `list` element, this is needed to parse a jsonapi valid payload. The second important aspect is the validation we are performing on the params. As you can see we are calling a params class method using a special DSL to add constraints to our input data. This is powered by [dry-validation](https://github.com/dry-rb/dry-validation). Take a look at the docs for more info on how to validate data, it's a really powerful tool.

## Conclusion

We have seen a very basic overview of the different building blocks that Hanami offers in the post. There is [A LOT](http://hanamirb.org/guides/) more in Hanami that I encourage everyone to investigate, community is really awesome and people are very helpful, you can find them on [Gitter](https://gitter.im/hanami/chat).

If you want to take a look at the source code you can find it here [bilby91/hanami-todo-app](https://github.com/bilby91/hanami-todo-app)

If you have any questions or want to explore different parts of Hanami please write a comment. Im just starting with the framework so looking into the different components of the project is something I will like to do.


Enjoy 🎉

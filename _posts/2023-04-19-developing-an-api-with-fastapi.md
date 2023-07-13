---
author: Oluwole Majiyagbe
header_img: https://testdriven.io/static/images/blog/fastapi/fastapi-graphql/social_card.png
source: testdriven
source_file: testdriven_2023-04-19-T16-50-50
source_url: https://testdriven.io/blog/fastapi-graphql/
subtitle: 'In this tutorial, you''ll learn how to build a CRUD app with '
tags:
- api
- fastapi
- testing
title: Developing an API with FastAPI and GraphQL

---
![Developing an API with FastAPI and GraphQL](https://testdriven.io/static/images/blog/fastapi/fastapi-graphql/social_card.png)


In this tutorial, you'll learn how to build a CRUD app with [FastAPI](https://fastapi.tiangolo.com/), [GraphQL](https://graphql.org/), and the [Masonite ORM](https://orm.masoniteproject.com/).


## Contents



* [Objectives](#objectives)
* [Why GraphQL?](#why-graphql)
* [Why Masonite ORM?](#why-masonite-orm)
* [Project Setup](#project-setup)
* [Masonite ORM](#masonite-orm)
	+ [Database Config](#database-config)
	+ [Masonite Models](#masonite-models)
	+ [Database Tables](#database-tables)
	+ [Table Relationships](#table-relationships)
* [GraphQL](#graphql)
* [Schema](#schema)
* [Mutations](#mutations)
	+ [Input Types](#input-types)
	+ [Add User Mutation](#add-user-mutation)
	+ [Remaining Mutations](#remaining-mutations)
	+ [Testing](#testing)
* [Queries](#queries)
* [Tests](#tests)
* [Conclusion](#conclusion)



## Objectives


By the end of this tutorial, you will be able to:


1. Explain why you may want to use GraphQL over REST
2. Use the Masonite ORM to interact with a Postgres database
3. Describe what Schemas, Mutations, and Queries are in GraphQL
4. Integrate GraphQL into a FastAPI app with Graphene
5. Test a GraphQL API with Graphene and pytest


## Why GraphQL?


(And why GraphQL over traditional REST?)


[REST](https://en.wikipedia.org/wiki/Representational_state_transfer) is the de-facto standard for building web APIs. With REST, you have multiple endpoints for each CRUD operation: GET, POST, PUT, DELETE. Data is gathered by accessing a number of endpoints.


For example, if you wanted to get a particular user's profile info along with their posts and relevant comments, you would need to call four different endpoints:


1. `/users/<id>` returns the initial user data
2. `/users/<id>/posts` returns all posts for a given user
3. `/users/<post_id>/comments` returns a list of comments per post
4. `/users/<id>/comments` returns a list of comments per user


This can result in request [overfetching](https://stackoverflow.com/a/44568365) since you'll probably have to get much more data than you need.


Moreover, since one client may have much different needs than other clients, request overfetching and underfetching are common with REST.


[GraphQL](https://graphql.org/), meanwhile, is a query language for retrieving data from an API. Instead of having multiple endpoints, GraphQL is structured around a single endpoint whose *return value is dependent on what the client wants instead of what the endpoint returns*.


In GraphQL, you would structure a query like so to obtain a user's profile, posts, and comments:



```
query {
  User(userId: 2){
    name
    posts {
      title
      comments {
        body
      }
    }
    comments {
      body
    }
  }
}

```

Voila! You get all the data in just one request with no overfetching since we specified exactly what we want.



> 
> FastAPI supports GraphQL via [Starlette](https://www.starlette.io/graphql) and [Graphene](https://graphene-python.org/). Starlette executes GraphQL queries in a separate thread by default when you don't use async request handlers!
> 
> 
> 


## Why Masonite ORM?


The Masonite ORM is a clean, easy-to-use, object relational mapping library built for the [Masonite](https://docs.masoniteproject.com/) web framework. It builds on the [Orator ORM](https://orator-orm.com/), an [Active Record](https://www.quora.com/Is-there-something-for-Python-like-ActiveRecord-in-Ruby-Rails/answer/Michael-Herman-3) ORM.


Masonite ORM was developed to be a [replacement](https://github.com/sdispater/orator/issues/369#issuecomment-633591842) to Orator ORM as Orator no longer receives updates and bug fixes.


It resembles other popular Active Record implementations, like Django's [ORM](https://www.fullstackpython.com/django-orm.html), Laravel's [Eloquent](https://laravel.com/docs/eloquent), AdonisJS' [Lucid](https://adonisjs.com/docs/4.0/lucid), and [Active Record](https://guides.rubyonrails.org/active_record_basics.html) in Ruby On Rails. With support for MySQL, Postgres, and SQLite, it emphasizes [convention over configuration](https://en.wikipedia.org/wiki/Convention_over_configuration), which makes it easy to create models since you don't have to explicitly define every single aspect. Relationships are a breeze and very easy to handle as well.


Although it's designed to be used in a Masonite web project, you can use the Masonite ORM with other Python web frameworks or projects.



> 
> For more on the Masonite ORM and how to use it with FastAPI, check out [Integrating the Masonite ORM with FastAPI](/blog/masonite-orm-fastapi/).
> 
> 
> 


## Project Setup


Create a directory to hold your project called "fastapi-graphql":



```
$ mkdir fastapi-graphql
$ cd fastapi-graphql

```

Create a virtual environment and activate it:



```
$ python3.11 -m venv env
$ source env/bin/activate

(env)$

```


> 
> Feel free to swap out virtualenv and Pip for [Poetry](https://python-poetry.org) or [Pipenv](https://github.com/pypa/pipenv). For more, review [Modern Python Environments](/blog/python-environments/).
> 
> 
> 


Create the following files in the "fastapi-graphql" directory:



```
main.py
requirements.txt

```

Add the following requirements to *requirements.txt* file:



```
fastapi==0.92.0
uvicorn==0.20.0

```

[Uvicorn](http://www.uvicorn.org/) is an [ASGI](https://asgi.readthedocs.io/en/latest/) (Asynchronous Server Gateway Interface) compatible server that will be used for standing up FastAPI.


Install the dependencies:



```
(env)$ pip install -r requirements.txt

```

In the *main.py* file, add the following lines to kick-start the server:



```
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def ping():
    return {"ping": "pong"}

```

To start the server, open your terminal, navigate to the project directory, and enter the following command:



```
(env)$ uvicorn main:app --reload

```

Navigate to <http://localhost:8000> in your browser of choice. You should see the response:



```
{
 "ping": "pong"
}

```

You've successfully started up a simple FastAPI server. To see the beautiful [docs](https://fastapi.tiangolo.com/features/#automatic-docs) FastAPI has for us, navigate to <http://localhost:8000/docs>:


![Swagger Docs]()


And <http://localhost:8000/redoc>:


![Redoc Docs]()


## Masonite ORM


Add the following requirements to the *requirements.txt* file:



```
masonite-orm==2.18.6
psycopg2-binary==2.9.5

```

Install the new dependencies:



```
(env)$ pip install -r requirements.txt

```

Create the following folders:



```
models
databases/migrations
config

```

The "models" folder will contain our model files, the "databases/migrations" folder will contain our migration files, and the "config" folder will hold our Masonite Database configuration file.


### Database Config


Inside the "config" folder, create a *database.py* file. This file is required for the Masonite ORM as this is where we declare our database configurations.



> 
> For more info, visit the [docs](https://orm.masoniteproject.com/installation#configuration).
> 
> 
> 


Within the *database.py* file, we need to add the `DATABASE` variable plus some connection information, import the `ConnectionResolver` from `masonite-orm.connections`, and register the connection details:



```
# config/database.py

from masoniteorm.connections import ConnectionResolver

DATABASES = {
  "default": "postgres",
  "mysql": {
    "host": "127.0.0.1",
    "driver": "mysql",
    "database": "masonite",
    "user": "root",
    "password": "",
    "port": 3306,
    "log_queries": False,
    "options": {
      #
    }
  },
  "postgres": {
    "host": "127.0.0.1",
    "driver": "postgres",
    "database": "test",
    "user": "test",
    "password": "test",
    "port": 5432,
    "log_queries": False,
    "options": {
      #
    }
  },
  "sqlite": {
    "driver": "sqlite",
    "database": "db.sqlite3",
  }
}

DB = ConnectionResolver().set_connection_details(DATABASES)

```

Here, we defined three different database settings:


1. MySQL
2. Postgres
3. SQLite


We set the default connection to Postgres.



> 
> Note: Make sure you have a Postgres database up and running. Feel free to change the default database connection.
> 
> 
> 


### Masonite Models


To create a new boilerplate [Masonite model](https://orm.masoniteproject.com/models#creating-a-model), run the following `masonite-orm` command from the project root folder in your terminal:



```
(env)$ masonite-orm model User --directory models

```

You should see a success message:



```
Model created: models/User.py

```

So, this command should have created a *User.py* file in the "models" directory with the following content:



```
""" User Model """

from masoniteorm.models import Model


class User(Model):
    """User Model"""

    pass

```


> 
> If you receive a `FileNotFoundError`, check to make sure that the "models" folder exists.
> 
> 
> 


Run the same commands for the Posts and Comments models:



```
(env)$ masonite-orm model Post --directory models
> Model created: models/Post.py

(env)$ masonite-orm model Comment --directory models
> Model created: models/Comment.py

```

Next, we can create the initial migrations:



```
(env)$ masonite-orm migration migration_for_user_table --create users

```

We added the `--create` flag to tell Masonite that the migration file to be created is for our `users` table and the database table should be created when the migration is run.


In the "databases/migration" folder, a new file should have been created:


`<timestamp>_migration_for_user_table.py`


Content:



```
"""MigrationForUserTable Migration."""

from masoniteorm.migrations import Migration


class MigrationForUserTable(Migration):
    def up(self):
        """
 Run the migrations.
 """
        with self.schema.create("users") as table:
            table.increments("id")

            table.timestamps()

    def down(self):
        """
 Revert the migrations.
 """
        self.schema.drop("users")

```

Create the remaining migration files:



```
(env)$ masonite-orm migration migration_for_post_table --create posts
> Migration file created: databases/migrations/2022_05_04_084820_migration_for_post_table.py

(env)$ masonite-orm migration migration_for_comment_table --create comments
> Migration file created: databases/migrations/2022_05_04_084833_migration_for_comment_table.py

```

### Database Tables


The `users` table should have the following fields:


1. Name
2. Email (unique)
3. Address (Optional)
4. Phone Number (Optional)
5. Sex (Optional)


Change the migration file associated with the Users model to:



```
"""MigrationForUserTable Migration."""

from masoniteorm.migrations import Migration


class MigrationForUserTable(Migration):
    def up(self):
        """
 Run the migrations.
 """
        with self.schema.create("users") as table:
            table.increments("id")
            table.string("name")
            table.string("email").unique()
            table.text("address").nullable()
            table.string("phone_number", 11).nullable()
            table.enum("sex", ["male", "female"]).nullable()
            table.timestamps()

    def down(self):
        """
 Revert the migrations.
 """
        self.schema.drop("users")

```


> 
> For more on the table methods and column types, review [Schema & Migrations](https://orm.masoniteproject.com/schema-and-migrations) from the docs.
> 
> 
> 


Next, update the fields for the Posts and Comments models, taking note of the fields.


Posts:



```
"""MigrationForPostTable Migration."""

from masoniteorm.migrations import Migration


class MigrationForPostTable(Migration):
    def up(self):
        """
 Run the migrations.
 """
        with self.schema.create("posts") as table:
            table.increments("id")
            table.integer("user_id").unsigned()
            table.foreign("user_id").references("id").on("users")
            table.string("title")
            table.text("body")
            table.timestamps()

    def down(self):
        """
 Revert the migrations.
 """
        self.schema.drop("posts")

```

Comments:



```
"""MigrationForCommentTable Migration."""

from masoniteorm.migrations import Migration


class MigrationForCommentTable(Migration):
    def up(self):
        """
 Run the migrations.
 """
        with self.schema.create("comments") as table:
            table.increments("id")
            table.integer("user_id").unsigned().nullable()
            table.foreign("user_id").references("id").on("users")
            table.integer("post_id").unsigned().nullable()
            table.foreign("post_id").references("id").on("posts")
            table.text("body")
            table.timestamps()

    def down(self):
        """
 Revert the migrations.
 """
        self.schema.drop("comments")

```

Take note of:



```
table.integer("user_id").unsigned()
table.foreign("user_id").references("id").on("users")

```

The lines above create a foreign key from the `posts`/`comments` table to the `users` table. The `user_id` column references the `id` column on the `users` table


To apply the migrations, run the following command in your terminal:



```
(env)$ masonite-orm migrate

```

You should see success messages concerning each of your migrations:



```
Migrating: 2022_05_04_084807_migration_for_user_table
Migrated: 2022_05_04_084807_migration_for_user_table (0.08s)
Migrating: 2022_05_04_084820_migration_for_post_table
Migrated: 2022_05_04_084820_migration_for_post_table (0.04s)
Migrating: 2022_05_04_084833_migration_for_comment_table
Migrated: 2022_05_04_084833_migration_for_comment_table (0.02s)

```

Thus far, we've added and referenced the foreign keys in our table, which have been created in the database. We still need to tell Masonite what type of relationship each model has to one another, though.


### Table Relationships


To define a one-to-many relationship, we need to import in `has_many` from `masoniteorm.relationships` within *models/User.py* and add it as decorators to our functions:



```
# models/User.py

from masoniteorm.models import Model
from masoniteorm.relationships import has_many


class User(Model):
    """User Model"""

    @has_many("id", "user_id")
    def posts(self):
        from .Post import Post

        return Post

    @has_many("id", "user_id")
    def comments(self):
        from .Comment import Comment

        return Comment

```

Do note that the `has_many` takes two arguments which are:


1. The name of the primary key column on the main table which will be referenced in another table
2. The name of the column which will serve as a reference to the foreign key


In the `users` table, the `id` is the primary key column while the `user_id` is the column in the `posts` table which references the `users` table record.


Do the same for *models/Post.py*:



```
# models/Post.py

from masoniteorm.models import Model
from masoniteorm.relationships import has_many


class Post(Model):
    """Post Model"""

    @has_many("id", "post_id")
    def comments(self):
        from .Comment import Comment

        return Comment

```

## GraphQL


While there are a number of [GraphQL libraries](https://fastapi.tiangolo.com/advanced/graphql/#graphql-libraries) that will work with FastAPI, [Strawberry](https://strawberry.rocks/docs/integrations/fastapi) is the [recommended](https://fastapi.tiangolo.com/advanced/graphql/#graphql-with-strawberry) library since it leverages data classes and type hints much like FastAPI.


Add `strawberry-graphql[fastapi]` to your *requirement.txt* file:



```
strawberry-graphql[fastapi]==0.158.0

```

Install:



```
(env)$ pip install -r requirements.txt

```

Next, update the *main.py* file like so:



```
import strawberry  # new
from fastapi import FastAPI
from strawberry.fastapi import GraphQLRouter  # new


# new
@strawberry.type
class Query:
  @strawberry.field
  def hello(self) -> str:
    return "Hello World"


schema = strawberry.Schema(Query)  # new
graphql_app = GraphQLRouter(schema)  # new
app = FastAPI()
app.include_router(graphql_app, prefix="/graphql")  # new


@app.get("/")
def ping():
    return {"ping": "pong"}

```

Start up your server:



```
(env)$ uvicorn main:app --reload

```

Navigate to <http://localhost:8000/graphql> . You should see the [GraphQL Playground](https://github.com/graphql/graphql-playground):


![Strawberry Playground]()


Type in a quick query to make sure all is well:



```
query {
  hello
}

```

You should see:



```
{
  "data": {
    "hello": "Hello World"
  }
}

```

## Schema


A [Schema](https://docs.graphene-python.org/en/latest/types/schema/) is the building block for every GraphQL application. They are used by GraphQL servers to describe the shape of the data. It serves as the core for the application gluing together all other parts, like Mutations and Queries.


Create the following three new files in the project root:


1. *schema.py* - this file will hold our Strawberry types. Strawberry supports code-first [schemas](https://strawberry.rocks/docs/general/schema-basics), which look a whole lot like Python data classes.
2. *controller.py* - will hold all our logics through which we perform database operations. The functions defined in this file will serve as our resolvers when we create our Mutations and Queries later on.
3. *core.py* - will tie it all together. This is where we'll define the *Query* and *Mutation* classes for read and write operations, which our GraphQL server can then perform.


In the *schema.py* file, let's define our types:



```
import strawberry

from typing import List, Optional


@strawberry.type
class CommentsType:
    id: int
    user_id: int
    post_id: int
    body: str


@strawberry.type
class PostType:
    id: int
    user_id: int
    title: str
    body: str
    comments: Optional[List[CommentsType]]


@strawberry.type
class UserType:
    id: int
    name: str
    address: str
    phone_number: str
    sex: str
    posts: Optional[List[PostType]]
    comments: Optional[List[CommentsType]]

```

With that, let's add the basic CRUD operations for creating, reading, updating, and deleting users, posts, and comments from the database.


## Mutations


[Mutations](https://docs.graphene-python.org/en/latest/types/mutations/) are used in GraphQL to modify data -- i.e., for creating, updating, and deleting data. We'll use a Mutation to create `User`, `Post`, and `Comment` objects and save them in the database.


### Input Types


Before we create the Mutation, we'll create a few Strawberry [input types](https://strawberry.rocks/docs/types/input-types). Input types make it easier for us to define the fields we want to use as input rather than passing them as arguments in our functions. To define an input type you can use the `strawberry.input` decorator.


Add the following input types to the *schema.py* file:



```
@strawberry.input
class UserInput:
    name: str
    email: str
    address: str
    phone_number: str
    sex: str

@strawberry.input
class PostInput:
    user_id: int
    title: str
    body: str


@strawberry.input
class CommentInput:
    user_id: int
    post_id: int
    body: str

```

### Add User Mutation


Now, in our *controller.py* class, let's add the logic to add a user. Create a mutate class, and in the class, include an `add_user` method, which takes an argument of type `UserInput`:



```
from models.User import User
from schema import UserInput


class CreateMutation:

    def add_user(self, user_data: UserInput):
        user = User.where("email", user_data.email).get()
        if user:
            raise Exception("User already exists")

        user = User()

        user.name = user_data.name
        user.email = user_data.email
        user.address = user_data.address
        user.phone_number = user_data.phone_number
        user.sex = user_data.sex

        user.save()

        return user

```

Now, to tie this class into our Mutation, add the following to *core.py*:



```
import strawberry

from controller import CreateMutation
from schema import UserType, PostType, CommentsType


@strawberry.type
class Mutation:
    add_user: UserType = strawberry.mutation(resolver=CreateMutation.add_user)

```

Now, we just have to add our Mutation into the `strawberry.Schema` instantiation. Open up the *main.py* file, and change the schema instantiation like so:



```
schema = strawberry.Schema(query=Query, mutation=Mutation)

```

Don't forget the import:



```
from core import Mutation

```

### Remaining Mutations


Add the other mutate methods to the *controller.py* file:



```
from models.Comment import Comment
from models.Post import Post
from models.User import User
from schema import CommentInput, PostInput, UserInput


class CreateMutation:

    def add_user(self, user_data: UserInput):
        user = User.where("email", user_data.email).get()
        if user:
            raise Exception("User already exists")

        user = User()

        user.name = user_data.name
        user.email = user_data.email
        user.address = user_data.address
        user.phone_number = user_data.phone_number
        user.sex = user_data.sex

        user.save()

        return user

    def add_post(self, post_data: PostInput):
        user = User.find(post_data.user_id)
        if not user:
            raise Exception("User not found")
        post = Post()
        post.title = post_data.title
        post.body = post_data.body
        post.user_id = post_data.user_id
        post.save()

        user.attach("posts", post)

        return post

    def add_comment(self, comment_data: CommentInput):
        post = Post.find(comment_data.post_id)
        if not post:
            raise Exception("Post not found")
        user = User.find(comment_data.user_id)
        if not user:
            raise Exception("User not found")

        comment = Comment()
        comment.body = comment_data.body
        comment.user_id = comment_data.user_id
        comment.post_id = comment_data.post_id

        comment.save()

        user.attach("comments", comment)
        post.attach("comments", comment)

        return comment

```

Update *core.py* as well:



```
@strawberry.type
class Mutation:
    add_user: UserType = strawberry.mutation(resolver=CreateMutation.add_user)
    add_post: PostType = strawberry.mutation(resolver=CreateMutation.add_post)
    add_comment: CommentsType = strawberry.mutation(resolver=CreateMutation.add_comment)

```

### Testing


Fire up Uvicorn again. Reload your browser, and within the GraphQL Playground at <http://localhost:8000/graphql>, execute the `addUser` Mutation:



```
mutation {
  addUser(userData:{
    name: "John Doe",
    email: "[[email protected]](/cdn-cgi/l/email-protection)",
    address: "My home address",
    phoneNumber: "1234567890",
    sex: "male"
  }){
    id
    name
    address
  }
}

```

You should get back a user object like this:



```
{
  "data": {
    "addUser": {
      "id": 1,
      "name": "John Doe",
      "address": "My home address"
    }
  }
}

```

Trying to add the same user with the same email again should now give you a list errors with the data key being null:



```
{
  "data": null,
  "errors": [
    {
      "message": "User already exists",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "addUser"
      ]
    }
  ]
}

```

Execute the `addPost` Mutation also to create a new post:



```
mutation addPost {
  addPost(postData: {
    userId: 1,
    title: "My first Post",
    body: "This is a Post about myself"
  })
  {
    id
  }
}

```

You should see:



```
{
  "data": {
    "addPost": {
      "id": 1
    }
  }
}

```

Finally, execute the `createComment` Mutation to create a new comment:



```
mutation createComment {
  addComment(commentData: {
    userId: 1,
    postId: 1,
    body: "Another Comment"
  })
  {
    id
    body
  }
}

```

## Queries


To retrieve data, we need to create a [Queries](https://strawberry.rocks/docs/general/queries) class, which we can then pass into the schema instantiation in the *main.py* file.


In *controller.py*, add a Queries class within which all functions will be our queries resolvers:



```
class Queries:

    def get_all_users(self) -> List[UserType]:
        return User.all()

```

Update the imports at the top:



```
from typing import List

from models.Comment import Comment
from models.Post import Post
from models.User import User
from schema import CommentInput, CommentsType, PostInput, PostType, UserInput, UserType

```

We've already defined our schema and our resolvers.


Now, let's tie it all together by updating *core.py* like so:



```
from typing import List, Optional  # new

import strawberry

from controller import CreateMutation, Queries  # updated
from schema import UserType, PostType, CommentsType


@strawberry.type
class Mutation:
    add_user: UserType = strawberry.mutation(resolver=CreateMutation.add_user)
    add_post: PostType = strawberry.mutation(resolver=CreateMutation.add_post)
    add_comment: CommentsType = strawberry.mutation(resolver=CreateMutation.add_comment)


# new
@strawberry.type
class Query:
    users: List[UserType] = strawberry.field(resolver=Queries.get_all_users)

```

Update *main.py*:



```
import strawberry
from fastapi import FastAPI
from strawberry.fastapi import GraphQLRouter

from core import Mutation, Query  # updated


schema = strawberry.Schema(query=Query, mutation=Mutation)
graphql_app = GraphQLRouter(schema)
app = FastAPI()
app.include_router(graphql_app, prefix="/graphql")


@app.get("/")
def ping():
    return {"ping": "pong"}

```

Start up your server:



```
(env)$ uvicorn main:app --reload

```

Navigate once again to <http://localhost:8000/graphql>, and execute the following query to return a list of users:



```
query getAllUsers {
  users{
    id
    name
    posts {
      title
    }
  }
}

```

Results:



```
{
  "data": {
    "users": [
      {
        "id": 1,
        "name": "John Doe",
        "posts": [
          {
            "title": "My first Post"
          }
        ]
      }
    ]
  }
}

```

To retrieve a single user, update the `Queries` class again, adding in the following resolver method:



```
class Queries:

    def get_all_users(self) -> List[UserType]:
        return User.all()

    # new
    def get_single_user(self, user_id: int) -> UserType:
        user = User.find(user_id)
        if not user:
            raise Exception("User not found")
        return user

```

Next, update the `Query` class in *core.py* like so:



```
@strawberry.type
class Query:
    users: List[UserType] = strawberry.field(resolver=Queries.get_all_users)
    get_single_user: UserType = strawberry.field(resolver=Queries.get_single_user)  # new

```

Try it out:



```
query getUser {
  getSingleUser(userId: 1) {
    name
    posts {
      title
      comments {
        body
      }
    }
    comments {
      body
    }
  }
}

```

The query should return a list of posts which in turn should contain a list of comments for every post object:



```
{
  "data": {
    "getSingleUser": {
      "name": "John Doe",
      "posts": [
        {
          "title": "My first Post",
          "comments": [
            {
              "body": "Another Comment"
            }
          ]
        }
      ],
      "comments": [
        {
          "body": "Another Comment"
        }
      ]
    }
  }
}

```

If you don't need the posts or comments, you can just remove the posts and comments block from the query:



```
query getUser {
  getSingleUser(userId: 1) {
    name
  }
}

```

Results:



```
{
  "data": {
    "getSingleUser": {
      "name": "John Doe"
    }
  }
}

```

Try with an incorrect user ID:



```
query getUser {
  getSingleUser(userId: 5999) {
    name
  }
}

```

It should return an error:



```
{
  "data": null,
  "errors": [
    {
      "message": "User not found",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "getSingleUser"
      ]
    }
  ]
}

```

Notice how exceptions are styled as messages.


## Tests


Graphene provides a [test client](https://docs.graphene-python.org/en/latest/testing/#test-client) for creating a dummy GraphQL client for testing a Graphene app.


We'll be using pytest so add the dependency to your requirements file:



```
pytest==7.2.1

```

We would also need the HTTPX library since FastAPI's TestClient is based on it. Add it to the requirements file as well:



```
httpx==0.23.3

```

Install:



```
(env)$ pip install -r requirements.txt

```

Next, let's create a separate config file for our tests to use so we don't overwrite data in our main development database. Inside the "config" folder, create a new file called test_config.py:



```
from masoniteorm.connections import ConnectionResolver


DATABASES = {
  "default": "sqlite",
  "sqlite": {
    "driver": "sqlite",
    "database": "db.sqlite3",
  }
}

DB = ConnectionResolver().set_connection_details(DATABASES)

```

Next, create a "tests" folder, and in that folder add a *conftest.py* file:



```
import pytest
from masoniteorm.migrations import Migration


@pytest.fixture(autouse=True)
def setup_database():
    config_path = "config/test_config.py"

    migrator = Migration(config_path=config_path)
    migrator.create_table_if_not_exists()

    migrator.refresh()

```

Next, add fixtures for creating a user, post, and comments:



```
@pytest.fixture(scope="function")
def user():
    user = User()
    user.name = "John Doe"
    user.address = "United States of Nigeria"
    user.phone_number = 123456789
    user.sex = "male"
    user.email = "[[email protected]](/cdn-cgi/l/email-protection)"
    user.save()

    return user


@pytest.fixture(scope="function")
def post(user):
    post = Post()
    post.title = "Test Title"
    post.body = "this is the post body and can be as long as possible"
    post.user_id = user.id
    post.save()

    user.attach("posts", post)
    return post


@pytest.fixture(scope="function")
def comment(user, post):
    comment = Comment()
    comment.body = "This is a comment body"
    comment.user_id = user.id
    comment.post_id = post.id

    comment.save()

    user.attach("comments", comment)
    post.attach("comments", comment)

    return comment

```

Don't forget the model imports:



```
from models.Comment import Comment
from models.Post import Post
from models.User import User

```

Your *conftest.py* file should now look like this:



```
import pytest
from masoniteorm.migrations import Migration

from models.Comment import Comment
from models.Post import Post
from models.User import User


@pytest.fixture(autouse=True)
def setup_database():
    config_path = "config/test_config.py"

    migrator = Migration(config_path=config_path)
    migrator.create_table_if_not_exists()

    migrator.refresh()


@pytest.fixture(scope="function")
def user():
    user = User()
    user.name = "John Doe"
    user.address = "United States of Nigeria"
    user.phone_number = 123456789
    user.sex = "male"
    user.email = "[[email protected]](/cdn-cgi/l/email-protection)"
    user.save()

    return user


@pytest.fixture(scope="function")
def post(user):
    post = Post()
    post.title = "Test Title"
    post.body = "this is the post body and can be as long as possible"
    post.user_id = user.id
    post.save()

    user.attach("posts", post)
    return post


@pytest.fixture(scope="function")
def comment(user, post):
    comment = Comment()
    comment.body = "This is a comment body"
    comment.user_id = user.id
    comment.post_id = post.id

    comment.save()

    user.attach("comments", comment)
    post.attach("comments", comment)

    return comment

```

Now, we can start adding some tests.


Create a test file called *test_query.py*.


Start by creating an instance of the `TestClient`:



```
from fastapi.testclient import TestClient

from main import app  # => FastAPI app created in our main.py file


client = TestClient(app)

```

Now, we'll add tests to:


1. Add a user
2. Get all users
3. Get a single user using the user's ID


Tests:



```
def test_create_user():
    query = """
 mutation {
 addUser(userData: {
 name: "Test User",
 email: "[[email protected]](/cdn-cgi/l/email-protection)",
 sex: "male",
 address: "My Address",
 phoneNumber: "123456789",
 })
 {
 id
 name
 address
 }
 }
 """

    response = client.post("/graphql", json={"query": query})
    assert response is not None
    assert response.status_code == 200

    result = response.json()
    assert result["data"]["addUser"]["name"] == "Test User"
    assert result["data"]["addUser"]["address"] == "My Address"

def test_get_user_list(user):
    query = """
 query {
 users {
 name
 address
 }
 }
 """

    response = client.post("/graphql", json={"query": query})
    assert response is not None
    assert response.status_code == 200

    result = response.json()
    assert type(result['data']['users']) == list
    assert result["data"]["users"][0]["name"] == user.name



def test_get_single_user(user):
    query = """
 query {
 getSingleUser(userId: %s) {
 name
 address
 }
 }
 """ % user.id

    response = client.post("/graphql", json={"query": query})
    assert response is not None
    assert response.status_code == 200

    result = response.json()
    assert type(result['data']['getSingleUser']) == dict
    assert result["data"]["getSingleUser"]["name"] == user.name

```

Run the tests:



```
(env)$ python -m pytest

```

This should execute all the tests. They all should pass:



```
=============================== test session starts ===============================
platform darwin -- Python 3.10.3, pytest-7.2.1, pluggy-1.0.0
rootdir: /Users/michael/repos/testdriven/fastapi-graphql
plugins: anyio-3.6.2, Faker-13.16.0
collected 3 items

tests/test_query.py ...                                                     [100%]

================================ 3 passed in 0.47s ================================

```

Following the same pattern, write tests for `Post` and `Comment` on your own.


## Conclusion


In this tutorial, we covered how to develop and test a GraphQL API with FastAPI, Strawberry, the Masonite ORM, and pytest. We covered how to create GraphQL Schemas, Queries, and Mutations. Finally, we tested our GraphQL API with pytest.


Grab the code from the [fastapi-graphql](https://github.com/testdrivenio/fastapi-graphql) repo.



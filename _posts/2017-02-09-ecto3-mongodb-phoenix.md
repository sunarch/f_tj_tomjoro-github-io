---
layout: post
title: Phoenix with Ecto and MongoDb
image:
tags: [Elixir, Phoenix, Ecto2, MongoDb]
---

How to use MongoDb with Phoenix and Ecto. 

## Background

Ecto 1.0 supported MongoDb. Ecto 2.0 & 3.0 don't. This has created a lot of confusion
for developers who want to use MongoDb with Phoenix web applications. It's
uncertain when, or if ever, MongoDb would be supported with Ecto >= 2

Also, there's not too much information explaining what's the best strategy.

In this blog you will learn:

* MongoDb works great with Elixir and Phoenix,
* It's not difficult, there are some nice ways to do it,
* How to do it,
* Example code for the most comment questions,

## Strategy

You want to build a Phoenix application that uses MongoDB. Here's how to do it.

1. Use the latest and greatest version of *Phoenix* and *Ecto 3*
2. Use the MongoDb driver directly (not the Ecto version)

Why Ecto? Ecto has a lot of great stuff, like Changesets, that are great.
And they work really well with MongoDb (I'll explain), but we're not
going to use the Repo part of Ecto.

One of the great things about Elixir and Phoenix/Ecto is that there really
aren't very many deep dependencies, so it's not difficult.

Here's what my mix.exs looks like. Your's may differ, but the point is
to get MongoDb and Ecto, but not the database specific Ecto drivers.

```elixir
defp deps do
  [{:phoenix, "~> 1.2.1"},
   {:phoenix_pubsub, "~> 1.0"},
   {:phoenix_html, "~> 2.6"},
   {:phoenix_live_reload, "~> 1.0", only: :dev},
   {:gettext, "~> 0.11"},
   {:cowboy, "~> 1.0"},
   {:mongodb, ">= 0.0.0"},
   {:poolboy, ">= 0.0.0"},
   {:phoenix_ecto, "~> 3.0"},
   {:httpoison, "~> 0.9.0"},
   {:uuid, "~> 1.1" },
   {:logger_file_backend, "0.0.8"},
   {:exrm, "~> 1.0.8" },
 ]
end
```

And the applications section:

```elixir
applications: [:phoenix, :phoenix_pubsub, :phoenix_html,
               :phoenix_ecto, :cowboy, :logger,
               :gettext, :mongodb, :poolboy, :logger_file_backend,
               :uuid, :httpoison]]
```

## Quick Start

### Configure databases
I added this to my prod.exs, dev.exs, test.exs:

In prod.exs:

```elixir
config :my_app, :db, name: "my_mongo_prod"
```
In dev.exs:

```elixir
config :my_app, :db, name: "my_mongo_dev"
```
In test.exs:

```elixir
config :my_app, :db, name: "my_mongo_test"
```

### Start MongoDb

In my lib/my_app.ex

```elixir
defmodule MyApp do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec


    # Define workers and child supervisors to be supervised
    children = [
      supervisor(MyApp.Endpoint, []),

      # 1. Start mongo
      worker(Mongo, [[database:
        Application.get_env(:my_app, :db)[:name], name: :mongo]])
    ]

    opts = [strategy: :one_for_one, name: MyApp.Supervisor]
    result = Supervisor.start_link(children, opts)

    # 2. Indexes
    MyApp.Startup.ensure_indexes   
    result
  end

  def config_change(changed, _new, removed) do
    MyApp.Endpoint.config_change(changed, removed)
    :ok
  end
end
```

1. Mongodb is started with the database pool.
2. I also take this opportunity to ensure any indexes I need.

My ensure_indexes function looks like this. You won't find a lot of
documentation for this! But 'command' is like a back door into MongoDb and
let's you do just about anything.

```elixir
defmodule MyApp.Startup do
  def ensure_indexes do
    IO.puts "Using database #{Application.get_env(:my_app, :db)[:name]}"
    Mongo.command(:mongo, %{createIndexes: "users",
      indexes: [ %{ key: %{ "email": 1 },
                    name: "email_idx",
                    unique: true} ] })
  end
end
```

## Ecto

### Changesets

Ecto uses the schema to validate the changeset. I use the Ecto schema
for fields I know I will always have. This is really handy.

If you have fields which are dynamic then the Schema will not really
help you in this case! But that's fine. I like to have a bit of both.

The trick to using the changeset is to realize that there is a `changes`
function that will give you _exactly the syntax you need to insert into
mongodb_! What could be sweeter.


### Schema

I can't include the whole thing, but enough so you get the idea:

```elixir
defmodule MyApp.User do
  use MyApp.Web, :model
  require Logger

  @primary_key {:id, :binary_id, autogenerate: true}  # the id maps to uuid
  schema "users" do
    field :email,         :string
    field :phone_number,  :string
    field :first_name,    :string
    field :last_name,     :string
    field :status,        :string
  end

  def changeset_new_user(user, params \\ %{}) do
    params = scrub_params(params)  # change "" to nil
    user
      |> cast(params, [:email, :phone_number, :first_name, :last_name])
      |> validate_required([:email])
      |> validate_phone_number
      |> put_change(:status, 1)
  end
```


## Example: Creating a New User

Here's an example of How I create a new users (in my Controller)

```elixir
defmodule MyApp.UserController do

...

  changeset_new_user = MyApp.User.changeset_new_user(%MyApp.User{}, params)

  {:ok, user} = Mongo.find_one_and_replace(:mongo, "users", %{}, changeset_new_user.changes, [return_document: :after, upsert: :true])
  Logger.info("created new user #{inspect(user)}")
```

## Example: Finding a Single (One) User

```elixir
cursor = Mongo.find(:mongo, "users",
              %{"email" => "someone@somewhere.com"}, limit: 1)
list = Enum.to_list(cursor)
if length(list) == 1 do
  {:ok, hd(list)}
else
  {:error, nil}
end
```

I've got a pull request in for a `find_one` version of this, so it may become
this in the near future :) :


```elixir
user = Mongo.find_one(:mongo, "users",
                %{"email" => "someone@somewhere.com"]})
```

## Example: Updating a Document

Here I just use $set to make the changes.

```elixir
 {:ok, user_after} = if Map.size(user_update_changeset.changes) >= 1 do
  Mongo.find_one_and_update(:mongo, "users",
    %{"email" => "someone@somewhere.com"},
    %{"$set" => user_update_changeset.changes},
    [return_document: :after])
```

## Example: Updating a Document no Changeset

For my applications it's like 80% of fields _are_ known ahead of time,
and I use Changesets for those.

The whole point of MongoDb is to have a flexible schema. That means you
will not know all the fields (key names) ahead of time. I think it's a good
strategy to separate your logic for the dynamic updates. Here's the code
I use.

This is a snippit from a controller that takes an id for the user
and a bunch of updates.

WARNING: The parameters are *not* validated. The service I have is working
internally with already validated! You probably want to validate things
before you stuff them in MongoDb.

```elixir
def put_client(conn, params) do
  user_id = params["id"]

  # need to merge to x the attributes
  attributes = params |> Map.delete("id")

  reduced = Enum.into(attributes, %{}, fn({key,value}) ->
    {"$.#{key}", value}
  end)

  {:ok, user} = Mongo.find_one_and_update(:mongo, "users",
    %{"email" => "someone@somewhere.com"},
    %{"$set" => reduced }, [return_document: :after])
```

## Example: Embedded Documents

Quite often you want to have an array of "models" that are embedded in a
document. This is how I did it.

I created another model object as above. Then I just use the changeset and
directly manage inserting it into an array like this:

```elixir
{:ok, result} = Mongo.find_one_and_update(:mongo, "users",
  %{"email" => "someone@somewhere.com"},
  %{"$push" => %{ addresses: one_address },
  [return_document: :after])
```

Removing is pull:

```elixir
Mongo.find_one_and_update(:mongo, "users",
  %{"email" => "someone@somewhere.com"},
  %{"$pull" => %{ addresses: one_address } } )
```

## Example: Find One and Update

In this case the actual fields are in my schema, so I can use a changeset and
changes.

```elixir
  user_update_changeset = MyApp.User.changeset_update_user(%MyApp.User{},
                                                              params)

  {:ok, user_after} = Mongo.find_one_and_update(:mongo, "users",
    %{"email" => "someone@somewhere.com"},
    %{"$set" => user_update_changeset.changes},
            [return_document: :after])
```



Note the "changes". Ecto has a struct and 'changes' gives you a map of the
changes.

## Bson Date Format

Here's a routine to get time_now into BSON format for MongoDb.

It's always a good idea to use Date formats in BSON format so that
MongoDB understands them. This will help when doing queries, etc.

```elixir
{% raw %}
defmodule MyApp.BsonTime do

  epoch = {{1970, 1, 1}, {0, 0, 0}}
  @epoch :calendar.datetime_to_gregorian_seconds(epoch)

  def from_erl_timestamp_to_ecto(erl_ts) do
    timestamp = from_os_timestamp_to_usec(erl_ts)
    ts = trunc(timestamp / 1000000)
    ts = ts + @epoch
    ts_tuple = :calendar.gregorian_seconds_to_datetime(ts)

    # now we need remainder
    us = rem(timestamp,1000) * 1000
    {{y, m, d}, {h, mn, s}} = ts_tuple
    {{y, m, d}, {h, mn, s, us}}
  end

  def from_os_timestamp_to_usec({megasecs,secs,microsecs}) do
    	(megasecs*1000000 + secs)*1000000 + microsecs
  end

  def bson_time_now() do
    now = :os.timestamp()
    now_ecto = from_erl_timestamp_to_ecto(now)
    BSON.DateTime.from_datetime(now_ecto)
  end
end
{% endraw %}
```

And if you have a BSON date returned from Mongo, you probably want to
render it in a view, or serialize it to JSON. Here's what I do:

```elixir
updated_at = BSON.DateTime.to_iso8601(user["updated_at"])
```

In my schemas, I didn't have a BSON.DateTime, so I specified a string.

```elixir
  field :created_at,   :string  
```

and use it like this:

```elixir
  user[:created_at] = MyApp.BsonTime.bson_time_now()
```

Since we are not using Ecto's persistence, you won't get the automatic
timestamp features of Ecto. In this case the easiest thing is just to
follow the pattern of using a business logic layer to do this.

_If we can make a Mongoid like driver, we could have automatic management
of the inserted_at, and updated_at, for example. But then again MongoDb's
Document _id has a timestamp already embedded in it._


## Mix Integrations

### Migrations

Just kidding! ;)   MongoDb doesn't have them.

Collections come into existence the instant you try to insert to one.
And fields are present on any document where you write one.

The only thing you should do on startup is ensureIndex (see above)

### Reset

I needed database reset and seed. The ecto mix tasks are not going to work,
so I just made my own.

```elixir
defmodule Mix.Tasks.MyApp.Reset do
  use Mix.Task

  @shortdoc "Resets all data in database"

  @moduledoc """
    There might be a better way to do this!

    You have to list all your collections listed.
  """

  def run(_args) do
    Mix.Task.run "app.start", []
    Mix.shell.info "Reset all data"
    Mongo.delete_many(:mongo, "users", %{})
    Mongo.delete_many(:mongo, "ponies", %{})
  end
end
```

This will not drop the indexes. So if you need to drop an index, you can
do it from the MongoDb console with db.users.dropIndexes()

### Seed

For seeding, again I just create a little mix task. Our devops guy hates
this because now we have to special case seeding the services that use
MongoDb instead of PostgreSQL. But it's just one line of code.

```elixir
  mix my_app.reset
  mix my_app.seed
```
and this

```elixir
defmodule Mix.Tasks.MyApp.Seed do
  use Mix.Task

  @shortdoc "Seeds "
  @moduledoc """

  """

  def run(_args) do
    Mix.shell.info "Seeding users -- starting application--"
    Mix.Task.run "app.start", []

    user = %{
      email: "someone@somewhere.com"
    }

    Mongo.insert_one(:mongo, "users", user)
  end
end
```

## Caveats

Don't mix atoms and string-keys in maps. The driver doesn't like that.

That's another reason to use Ecto Changesets, everything will end up
as atoms, even if you have string keys.

Otherwise resort to walking the Map and getting it all fixed. There
are examples in Phoenix on how to Enum a Map and convert atoms to strings
and vice-versa.

## Conclusion

The MongoDb drivers for Elixir have proven very reliable. It also support
replica sets now. And it also works well with Phoenix.

All you need is a little bit of configuration and you'll be good to go.

If you liked this article, have some corrections or additions, send me an
email or pull-request. I'm sure other developers have done some similar
solutions.

# Proposal

It would be great if there were something akin to Rails's Mongoid drivers
in Elixir/Phoenix.

I'd love to see a Mongoid-like driver for Phoenix. It would grab the
best parts of Ecto (or be a branch) and provide handy features for dealing
with the various times, dates, timesamps, ids etc., and embedded documents.

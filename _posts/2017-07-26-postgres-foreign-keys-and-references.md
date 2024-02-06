---
title: Postgres Foreign Keys and References
layout: post
date: 2017-07-26
author: andrew
categories: [ rails ]
---

Few things in the world are more frustrating to a developer as when you push your code to Heroku, and it does not work. Why? Because you changed databases from SQLite3 to PostgreSQL.

While writing my latest Ruby on Rails application, I ran into this problem when trying to migrate my database on Heroku:

```
StandardError: An error has occurred, this and all later migrations canceled:                                               
PG::UndefinedTable: ERROR:  relation "departure_airports" does not exist     
                  
: CREATE TABLE "flights" ("id" bigserial primary key, "departure_airport_id" bigint, "arrival_airport_id" bigint, "take_off" timestamp, "duration" time, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL, CONSTRAINT "fk_rails_b88a099d1c" 
                      
FOREIGN KEY ("departure_airport_id")                         
  REFERENCES "departure_airports" ("id"), 
  CONSTRAINT "fk_rails_91bb366993"                       
FOREIGN KEY ("arrival_airport_id")                         
  REFERENCES "arrival_airports" ("id"))
```

But… but.. everything was working locally using SQlite3. What’s the problem?
The problem lies within the migration file used for creating the relationships between the flight and airport models. This is what I had:

```
class CreateFlights < ActiveRecord::Migration[5.1] 
  def change
    create_table :flights do |t|
      t.references :departure_airport,  foreign_key: true
      t.references :arrival_airport,    foreign_key: true
      t.datetime :take_off
      t.time :duration
      t.timestamps                           
    end
  end
end
```

What’s the problem? The lines `t.references`

Great, we’ve found the problem. Now how do I fix it?

The good news is that this is easily fixed. Instead of using `.references` we will instead create the foreign keys and indexes ourselves.

To do this, we will change `t.references` to `t.integer` and change the column name from `:departure_airport` to `:departure_airport_id`

It is then necessary to add the foreign key and index to create the relationship within the database. We do this by adding
```
add_foreign_key :flights, :airports, column: :departure_airport_id
add_index :flights, :departure_airport_id
```
Repeat the same process for :arrival_airportto give us a final migration file of:

```
class CreateFlights < ActiveRecord::Migration[5.1]
  def change
    create_table :flights do |t|
      t.integer :departure_airport_id
      t.integer :arrival_airport_id
      t.datetime :take_off
      t.time :duration
      t.timestamps
    end
    add_foreign_key :flights, 
                    :airports, 
                    column: :departure_airport_id
    add_index :flights, :departure_airport_id
    add_foreign_key :flights, 
                    :airports, 
                    column: :arrival_airport_id
    add_index :flights, :arrival_airport_id
  end
end
```
Push your new code to Heroku and your migrations should now work

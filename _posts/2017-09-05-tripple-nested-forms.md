---
layout: post
title: Triple Nested Forms in Rails 5
date: 2017-09-05
image: "assets/images/nesting.jpg"
author: andrew
categories: [ rails, "web development" ]
---

There are countless videos out there showing you how to make a single nested form. That is, a form that accepts two models. This tutorial is different in that it shows you how to make a 3+ nested form. Let’s get started.
First, we’ll make a new rails app, `$ rails new nested-forms` and enter into the directory.
To complete the nesting, we will be using three models, show, season, and episode. Each show has many seasons which have many episodes

Let’s generate the first model:

```
nested-forms $ rails g scaffold show name:string
```

then our second model, season:

```
nested-forms $ rails g model season number:integer show:references
```

Note how we include a reference to show, it will be included in our migrations.
Now let’s edit our models to apply the associations:

```
# models/show.rb
class Show < ApplicationRecord
  has_many :seasons
  accepts_nested_attributes_for :seasons
end
```

```
# models/season.rb
class Season < ApplicationRecord
  belongs_to :show, optional: true
end
```

and then add to our controller:

```
# controllers/shows_controller.rb
def new
  @show = Show.new
  @show.seasons.build
end

...

private
def show_params
  params.require(:show).permit(:name, 
    :seasons_attributes => [:number]
  )
end
```

and finally, let’s update our form to render these fields:
```
<%= form_with(model: show, local: true) do |form| %>
  # Show name and label
  <%= form.fields_for :seasons do |s| %>
    <%= s.label :number %>
    <%= s.number_field :number %>
  <% end %>
  # Submit Button
<% end %>
```

Notice how we chain `fields_for` onto form and then pass our new variable s into the block. This is what creates the association.

We’ve now completed our first nest. Try it out. Fire up the server, go to `/shows/new` and create a new show with a title and number (the number won’t show up after creation because we haven’t added it to the show view)

## Triple Nesting

Now the fun begins. Triple nesting.

Let’s create a new model:

```
nested-forms $ rails g model episode title:string season:references
```

Similar to to how seasons referenced show, our new model references seasons.

To get the tri-nested form inside the nested form, we basically have to repeat the procedure as above, but this time inside of season

First, we edit the models:

```
# models/season.rb
class Season < ApplicationRecord
  belongs_to :show, optional: true
  has_many :episodes
  accepts_nested_attributes_for :episodes
end
```
```
# models/episode.rb
class Episode < ApplicationRecord
  belongs_to :season, optional: true
end
```

then edit the shows_controller to accept further nested attributes:
```
# controllers/shows_controller.rb
  def new
    @show = Show.new
    @show.seasons.build.episodes.build
  end
private
  def show_params
    params.require(:show).permit(:name, 
      :seasons_attributes => [:number,
        :episodes_attributes => [:title]
      ]
    )
  end
```

and then, finally, the form:

```
<%= form_with(model: show, local: true) do |form| %>
# Show name and label
  <%= form.fields_for :seasons do |s| %>
    <%= s.label :number %>
    <%= s.number_field :number %>
    <%= s.fields_for :episodes do |e| %>
      <%= e.label :title %>
      <%= e.text_field :title %>
    <% end %>
  <% end %>
# Submit Button
<% end %>
```
Notice how the episodes fields for is nested inside of the seasons fields for. We use the symbol (`s`) from the seasons to create the fields for episodes.

Now, migrate the changes. When you fire up the server and go to `/shows/new` you should see three fields, a show, a number, and a title. Try submitting a new record and examine it in rails console and you should see that it is accepting all records.

To keep nesting, just repeat the above instructions again until you pass out.

## Conclusion
I hope that you find this article useful for developing rails applications. If you have any questions or comments feel free to leave a comment below, or tweet at me: @andyduss

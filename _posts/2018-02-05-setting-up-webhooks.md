---
title: Setting up Github Web Hooks
date: 2018-02-05
layout: post
author: andrew
categories: [ automation, ruby ]
image: assets/images/webhooks.jpg
---

Setting up Webhooks with Github is a simple way to start many common tasks when deploying a website. In this tutorial I will be showing you how to create an automated script to deploy your Jekyll blog when a merge occurs on a specified branch.

### Overview

In this tutorial, we will be using an existing Jekyll blog and automating the process of building the site when a new change is merged into the `deploy` branch on GitHub.  To do this we will need the following:

* An existing Jekyll Blog running on a server
* Access to `sudo` user and the blog repository on Github
* A willingness to learn

From a high level, we can explain the process as the following:

1. We make changes to the blog (write a new post) and push to Github's `master` branch
2. We make and accept a Pull Request (PR) from `master` to `deploy`
3. Github sends a webhook HTML message to our server, which is listening for this message.
4. Our server checks that the change was made to the `deploy` branch
5. Our server pulls the change
6. Our server rebuilds the Jekyll blog via the command `jekyll build`

All code for this tutorial can be found [here](https://github.com/mindovermiles262/jekyll-webhooks)

### Getting Started

The first thing we need to do is set up a web server on our host to listen for any Github webhooks.  We can use a simple [Sinatra](http://sinatrarb.com) server to perform this.

Inside of a new `webhooks` folder, let's create a new file, `server.rb`

```ruby
# webhooks/server.rb
require 'sinatra'
require 'json'

get '/testing' do
  puts "Sinatra Server Listening"
  "<h1>Sinatra Server Listening</h1>"
end

```

Save and exit, then start the server with `ruby server.rb -o 0.0.0.0 -p 6789` specifying the host (`-o`) and the port (`-p`).  If you receive a warning about Sinatra not being installed, install it with `gem install sinatra`

Now open your web browser and go to `http://localhost:6789/testing` and you should see a message, "Sinatra Server Listening." You should also see the same message in your terminal.  Our web server is now working and listening!

Now that our server is running we can configure it to listen for the webhooks.  Let's create a new route on the server for this purpose:

```ruby
# webhooks/server.rb
require 'sinatra'
require 'json'

get '/testing' do
  puts "Sinatra Server Listening"
  "<h1>Sinatra Server Listening</h1>"
end

post '/update-blog' do
  puts "Update blog received"
end
```

With that, we can set up our Github Webhooks.

### Configuring Github
The next step is to enable webhooks.  Webhooks are simply HTTP messages sent to our server each time an action occurs on Github. They are highly customizable and are very useful. For this project we will be using the default settings.

Go to your Jekyll blog repository on Github and then go into the Settings menu.  Select the "Webhooks" tab from the side and click "Add webhook"

In the `Payload URL` field, enter your server's IP address, followed by the port you want to use and route. For example, if my server IP address was 123.123.123.123 and I was using port 6789, my URL would be `http://123.123.123.123:6789/update-blog`

Change the content type to `application/json` so the message content is sent in a nice JSON format.

With that complete we can click "Add webhook" and test it out.

### Testing our Webhook

With the Webhook created, go back into it by selecting the "Edit" button. At the bottom of the page you should now see a "Recent Deliveries" section. Click on the only delivery notification to expand it. If everything has been set up properly, the delivery should be successful (Response 200).  If you go back to your server, you should see a new line with the content "Update blog received". If not, you will need to ensure that your server is running and accessible to the world (Check your firewall and port forwarding).

### Automagically building on update
Now that our server is running and accepting webhook messages, we can start automating the build process.  In a typical setup, jekyll blogs are built by issuing the `jekyll build` command from the root directory of the blog.  Let's make a new shell script, called `update-blog.sh` to do this for us:

```bash
#!/bin/bash

echo "Changing Directories"
cd /home/your-name/www/blog
echo "Pulling Deploy Branch"
git pull origin deploy
echo "Building Jekyll"
jekyll build
```

Test the above script by running `./update-blog` in your terminal.

We now set our server to run the update-blog script when a webhook is received:

```ruby
# webhooks/server.rb
require 'sinatra'
require 'json'

get '/testing' do
  puts "Sinatra Server Listening"
  "<h1>Sinatra Server Listening</h1>"
end

post '/update-blog' do
  system('/home/your-name/webhooks/update-blog.sh')
  puts "Blog Updated"
end
```

Test it out by going to your Github repo's webhooks page and clicking "Redeliver." Your server should now accept the message and update your blog:

```bash
Changing Directories
Pulling Deploy Branch
From github.com:your-github-name/blog
 * branch            deploy     -> FETCH_HEAD
Already up-to-date.
Building Jekyll
Configuration file: /home/your-name/www/blog/_config.yml
            Source: /home/your-name/www/blog
       Destination: /home/your-name/www/blog/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 1.132 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
Blog Updated
```

Success! You now have an auto-updating blog!

### Listening for specific events

That's great! We now have an auto-updating blog, but what if we want to write a draft post and don't want to deploy it?  We can do that using branches. We can set up our Sinatra server to check of the push was made to the `deploy` branch. If it was we will update the blog. If not, we won't do anything.  Let's make this change now.

Each time Github sends a webhook, it will send some information along with it.  We can print this information by adding to our `server.rb` file:

```ruby
...
post '/update-blog' do
  # system('/home/your-name/webhooks/update-blog.sh')
  # puts "Blog Updated"
  puts JSON.parse(request.body.read)
end
```

The branch information we are looking for should be the first data to be sent: 

```bash
{"ref"=>"refs/heads/deploy" ...
```

We can add a simple `if` statement to our Sinatra server to check if we posted to the `deploy` branch:

```ruby
# webhooks/server.rb
require 'sinatra'
require 'json'

get '/testing' do
  puts "Sinatra Server Listening"
  "<h1>Sinatra Server Listening</h1>"
end

post '/update-blog' do
  if JSON.parse(request.body.read)["ref"] == 'refs/heads/deploy'
    system('/home/your-name/webhooks/update-blog.sh')
    puts "Blog Updated"
  else
    puts "Not pushed to 'deploy' branch"
  end
end
```

Now if you push to your `master` branch you should receive a message in your server terminal saying "Not pushed to 'deploy' branch". If you then create a PR and merge the changes from `master` into `deploy`, your blog should automatically update.  Pretty simple.

### Starting Server at System Start

The last thing we need to do is to start the Sinatra server at system start. This will ensure that, should our server crash and restart, our Sinatra server will restart and be able to update our blog without having to log in.

We will be using a handy program called `tmux` to do this.  Tmux is a program that allows terminals to be 'detached' (run in the background) and continue without the terminal window being open. Let's write a cron script (named `start-server.cron`) to start our Sinatra server.

```bash
#!/bin/bash

/bin/sleep 5

/usr/bin/tmux new-session -d -s webhooks-server
/usr/bin/tmux send -t webhooks-server "ruby /home/student/webhooks/server.rb -o 0.0.0.0 -p 1989" C-m
```

In this script, we create a new tmux session and call it `webhooks-server`. We then tell the instance to type `ruby /home/your-name/webhooks/server.rb -o 0.0.0.0 -p 6789` and then hit enter (`C-m`).  Test to make sure you didn't make any spelling errors by running `./webhooks-server.cron`. You can check to make sure it's working by attaching the tmux session with `tmux a`. You should see the typical Sinatra server output. To detach the tmux session (keep running in background) you will need to type `CTRL` - `b` + `d`, or to close the tmux session, just `CTRL` - `d`.

With the script working, we can enable it to start at system start by adding it to our crontabs. From your terminal:

```bash
$ crontab -e
```

then add the following at the bottom of the file:

```bash
@reboot /home/your-name/webhooks/start-server.cron
```

That's it. You now have an automatic build process for your blog! Well done!

### Conclusion

I hope that this tutorial has taught you something about the use of Github webhooks.  They are very useful in the world of automation and are highly customizable.  You can add specific event triggers to make them even more customizable.  If you have any questions please feel free to reach out to me at the email above!
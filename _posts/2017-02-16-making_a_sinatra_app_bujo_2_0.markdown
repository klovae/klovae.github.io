---
layout: post
title:  "Making a Sinatra App: BuJo 2.0"
date:   2017-02-16 04:51:06 +0000
---

For my Sinatra portfolio project, I built a digital version of the popular [Bullet Journal](http://bulletjournal.com/), a to-do list and journaling methodology that bills itself as "The analog system for the digital age". 

Without going too far down the Bullet Journal (BuJo for short) rabbit hole, there are some basics to know to understand how the app works and why I made certain decisions.

At its very base, bullet journaling is a fancy to-do list system. Every day, you write your tasks down in a list with a bullet for each task. This is your "daily log". When you finish a task, you turn that task's bullet into an "X". If you can't get to it, you do what's called "migrating" -- you move the task to the next day by turning that original bullet into an arrow (">") and re-write the task in tomorrow's log.

For a working example, here's my log from today:

![](http://i.imgur.com/CYxYKOT.jpg)

Now don't get me wrong, I really _like_ the analog aspect of keeping a Bullet Journal. I'm an office supply nerd who used to confine their nerd-dom strictly to pens, but in the last couple years I've started expanding my domain to include nice notebooks. I also like the freedom that comes with dot-grid paper, which makes it simple to track everything from scribbled business card ideas to my (lack of) progress with working out. It's easy to cross tasks out, move them around, add new ones, and complete them.

But what you lose with an analog system is an easy way to look at your data. And with a digital Bullet Journal, you could compare weeks in terms of tasks added, completed, and migrated, or even set alerts for yourself when you've migrated a task one too many times (there are only so many times you should put off making that appointment). In some ways I do some of these things through weekly check-ins with myself, but it's time consuming and doesn't really let me see the whole picture. I'd love to be able to look at a full month of days and start making connections like "wow when I meditate I definitely get a lot more done" -- and realizing sometimes that a new habit that I thought was 100% going to turn my life around, is actually making me less productive. 

The first step to being able to get all this data was to build a working BuJo app, that followed the same set of rules and had the same features as an analog BuJo. There are other components to a full Bullet Journal, like a monthly log, an index, and of course notes, but for the kinds of data that I'm looking to collect, the daily log is really the most important, so that's what I set out to build.

## GETTING STARTED BETTER

In a lot of ways, my Sinatra project feels like my first Real dev project -- unlike our first assessment project, the CLI gem, the Sinatra project requires an actual factual front end, user accounts, and a database. Knowing how much work (and sweat, and tears) it took to put together my CLI gem, I was determined to make use of [the lessons I learned with that first project](http://elyseklova.com/2016/09/26/making_a_gem_podcast_finder/).

A couple of the biggest lessons from my CLI gem were really important to making working on this project a lot smoother and easier on myself:

**If there’s a framework to help me get started, I should use it because there’s probably a reason someone created it.**

For the CLI gem, I stubbornly built the program first and then tried to make it work as a gem. While using the right scaffolding wasn't as big a deal with this, I got started with (Learn's very own) Brian Emory's Sinatra scaffolding gem, [Corneal](https://github.com/thebrianemory/corneal). It helped me get set up quickly with the right files and folders in the right places.

**The preparation for the project is more important than starting the project AND adequate preparation (i.e. reading available resources) can save a lot of time over the course of a project even if it seems like it’s delaying the start.**

I spent a _lot_ of time prepping for this project. With a domain like Bullet Journaling, I knew there were going to be some tricky things to figure out, especially how to migrate tasks, and so I did a ton of brainstorming and diagramming to make sure my models fit together correctly and that the tables in my database worked accordingly. That's not to say I didn't end up creating a bunch of additional migrations to change things later, but getting up and running with my initial database and models went really smoothly.

**I have the ability to solve any coding issue/challenge that I come up against BUT just because I can solve an issue myself doesn’t mean it’s necessarily a good use of time**

I am proud to say that I did a _much_ better job asking for help with this project. I came up against a couple of points where I wasn't sure what best practice would be (for example, having task-related routes that also required day ids), and instead of making something up, I talked to the instructors and it helped immensely.

**Just because other people finish the same project in less time doesn’t mean I’m failing.**

This one ended up being hugely important to me, because honestly, finishing this project took a while. I watched my Fwitter partner finish her Sinatra app and go sailing into Rails before I even finished all the functionality of my Sinatra project. Between holiday travel, starting the new year, and picking up consulting work for the first time since August, I haven't been as focused as I would have liked to be on this project. 

That said, I don't want to ignore a lot of the benefits that came with taking time between work sessions on this project. Figuring out new functionality and fitting it into my app was significantly easier with project, I think in large part because most of the solutions I came up with happened when I wasn't slogging away in front of my text editor. Logically I know that this is normal, but up to this point I had a lot of trouble walking away. Now I've seen firsthand how much better a project goes when I give my brain a little bit of space to breathe and work on other things.

**Rather than starting with a huge scope and cutting it later, start with a smaller scope and then add.**

For me, this lesson in particular was the biggest gamechanger. With the CLI gem project, I was already planning 4 different major sets of functions that I wanted that app to be able to do before I even figured out that I could actually get and use the data required for all those functions. This time, I broke the project into pieces, and kept myself from moving to the next piece until the current one was built and functioned correctly:

1. Build the basic to-do list app that let a user take CRUD actions on the tasks, and that associated tasks with days.
2. Add "Events" (a Bullet Journal function) that would let the user record memorable things that happened on a specific day, like seeing a good movie, eating at a new restaurant, or running into an old friend. Events have basically the same functionality as tasks, with the exception that they don't get checked off.
3. Let the user change their own settings (for example, their associated name, email, and password)
4. Added migration functionality

Following this process almost completely eliminated the feelings of being stuck and overwhelmed. It also made identifying bugs, errors, and general weird behavior significantly easier because I always knew that all the functionality I'd built up to that point was working correctly.

## MODELS AND MIGRATIONS

For the most part, designing this app was fairly straightforward; I started with three basic models, User, Day, and Task. A day `belongs_to` users and tasks belong to both days and users. A user `has_many` of both. A user will create tasks on a specific day (usually today), and then can see all the tasks for a specific day, plus edit and delete any of those tasks. To "complete" a task, the user is actually editing the status of that task with a `patch` request, changing it from "open" to "complete" Once I got tasks working, I added Events (very similar to tasks but without the option to complete or migrate)

The concept that sets the Bullet Journal methodology apart is migrating your tasks. It's a great way to make sure important things don't fall off your radar, and just annoying enough that it encourages you to really evaluate if a task is important if you just keep migrating it. 

In an analog Bullet Journal, migration is pretty simple. You have a task listed today that you know you're not going to finish, so you change the bullet to ">" and just rewrite it the next day. But moving into the digital realm of models and associations, things get a little more complex. 

Task migration needed to fulfill the following requirements:

* A user can migrate a task to the current day
* A user can migrate a task as many times as they want
* The user can see their migrated task on the date it was migrated from
* A task that has been migrated is indicated by ">" on the original date(s) it was migrated from
* The user can also see the migrated task on the new day it has been migrated to
* On the new date, the migrated task shows up as normal, with a bullet, and can be completed
* Tasks migrated from the past automatically move to the current day
* Tasks migrated from the current day move to "tomorrow"
* For future functionality, there needs to be a way to track how many times a task has been migrated

With the same task at two different "states" ("open" vs "migrated") at the same time, I couldn't come up with a good way to integrate the migration functionality into the task model itself. Plus the requirement that a task can be migrated indefinitely made it impossible -- each new migration would need a column to keep track of all the original dates.

That need for a infinite new columns is what eventually got me to the idea of a Migration model -- basically a join table that would create an entry for each migration, and track the task_id, the original day_id, and the new day's id. Then from the user side, the user would essentially submit a patch request that would change the task's day_id to the current day's ID, and create the migration.

Here's what the route looks like on the back end:

```
  post "/tasks/:taskid/migrate" do
    task = Task.find_by_id(params[:taskid])
    day_id = task.day.id
    if task.day.date == Date.today
      new_day = Day.find_or_create_by(date: Date.tomorrow, user: current_user)
    else
      new_day = Day.find_or_create_by(date: Date.today, user: current_user)
    end
    migrate = Migration.create(task_id: task.id, day_id: day_id, new_day_id: new_day.id)
    task.day = new_day
    task.save
    flash[:success] = "Task migrated"
    redirect "/days/#{day_id}"
  end
```

The above code works together with the ERB to display tasks on a given day -- in addition to displaying open and completed tasks for that day, the enumerator also checks for migrations belonging to that day, and if it finds them, it displays each one with the task's content.

```
<% if @day.migrations %>
  <% @day.migrations.each do |migration| %>
    <div class="task">
      <div class="migrated icon task-component"></div>
      <div class="task-component">
        <%= migration.task.content %>
      </div>
    </div>
  <% end %>
<% end %>
```

Honestly I'm super pleased with how this solution turned out; it fulfills all the requirements, and even provides opportunities for future features like counting how many times a task has been migrated (`task.migrations.count`), or checking how many times overall a user has migrated tasks (`user.migrations`), or how many tasks have been migrated from a given day (day.migrations). 

From there you can give the user access to stats like ratio of completed tasks to migrated tasks, days of that have the best ratio of completed tasks, days that have the most migrations -- all of which can help the user adjust their work behavior. If you have consistently more migrations than completions, maybe you're overloading yourself or overpromising a boss? Or maybe you're slacking too much after lunch on Wednesdays?

I'll have to save that for future versions though. In the more immediate term, the next goal is to pass my assessment and move on to Rails.








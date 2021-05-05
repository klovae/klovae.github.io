---
layout: post
title:  "Iterating Forward: A Ruby App 5 Years Later"
date:   2021-05-04 13:51:07 -0500
---

It feels unreal to be writing a post for this blog 5 years to the date after [def greeting](http://elyseklova.com/2016/05/04/def_greeting.html). In the time since I started Flatiron School's Self-Paced Software Engineering Program:

* I moved from Houston, TX to Charleston, SC
* I moved from Charleston back to my hometown of choice, Atlanta
* I started a [leadership development company](https://blazeleadership.co) with my former boss from my first-ever job
* We created and launched a leadership assessment that I [coded myself](https://trailblazerstest.co) (thanks to Flatiron)!
* My partner and I bought our first house in the middle of a pandemic
* Said pandemic swept through like a literal plague and changed everything about how we live and work

When I started this program, I was coming off a job in the nonprofit sector and I wasn't sure what was next for me. I knew I wanted to make things, and learning software development seemed like a really smart move towards that end. And even though my primary work is with my company, knowing how to code has become more important than ever, for my own satisfaction as well as the benefit of my firm.

To that end, let's get to talking about my Rails Project.

## FRONT END NOTES

In a lot of ways, my Rails project builds on the learning from my [Sinatra Project](http://elyseklova.com/2017/02/16/making_a_sinatra_app_bujo_2_0.html). I built a digital version of the bulleted to-do list system that has become ubiquitous in the last 5 years or so. While I was getting back on the horse with coding and solidifying my knowledge of Rails, I actually [fully redid my Sinatra app](https://github.com/klovae/bujo-3-point-0) in Rails before moving on to my official project domain, a light project management system. Meet Tandem:

![](https://i.imgur.com/hxuaIkf.png)

I love how design can make a thing feel Legitimate long before the functionality is there to support it, and it gave me a lot of energy to work on the rest of the functionality; I wanted the back end to match the front end. Not to say that I started with fully built-out views -- my goal from the start was to get basic project and task functionality working first. So before we got fancy, we got Times New Roman:

![](https://i.imgur.com/D0nLK6B.png)

I'm pleased with how the final layout came together; there's no framework undergirding it, just CSS `grid-template-areas`:

![](https://i.imgur.com/R0csnsa.png)

## BACK END NOTES
Tandem is primarily an extension/improvement on the functionality I built for my Sinatra app. It is a project management tool that handles CRUD actions with tasks, with several additions to 1) make the app more functional for real-world use and 2) fulfill the assessment requirements. Tandem includes several major additions:

* Ability to create projects as a way to manage groups of tasks
* Ability to create project sections as a way to organize work
* Ability to share projects with other users, including a fairly robust invitation system
* Ability to control permissions for invited users, with functionality restricted for non-owner users

Here is the model diagram that makes that functionality possible:

![](https://i.imgur.com/PyTj2Bo.png)

Originally I had plans for nesting tasks under other tasks using a self-join, but that's something I can always explore later. The underlying structure is there in the model and database, but the user interface doesn't make the functionality available.

I would say the highlight of this project for me as far as stretching my abilities was creating the project sharing/permissioning functionality. While this app does not make use of Rails mailers, I wanted to at least approximate a familiar process for users, so I worked against the following user stories:

* As a project owner, I can invite another Tandem user to my project by entering the email associated with their account
* As a project owner, I can adjust the degree of control that an invited user has over my project
* As a Tandem user, I can receive and view invitations to other Tandem users' projects
* As a Tandem user, I can accept invitations to other Tandem users' projects
* As a project owner, I can create assign tasks in my project to other users who are on my project
* As a project manager, I can create and assign tasks in my project to other users who are on my project
* As a project contributor, I can complete tasks that were assigned to me

### Invitations

Without getting into Rails ActionMailer, I decided to implement a stripped-down invitation system. Of course the ideal would be that a user could invite a non-Tandem user to participate in their project via email and that non-User could create an account to then accept the invitation and join the project. But that is very far past the scope of this assessment application. Instead, I created a Permissions index view for projects that shows the list of collaborators and contains a form to invite new collaborators. When project owners invite new collaborators, their status shows up as pending from the Permissions view:

![](https://i.imgur.com/qSYFBhU.png)

Then, when the invited user signs into their account or goes to their home screen, they will see the invitation waiting for them:

![](https://i.imgur.com/v8KybCd.png)

If the user accepts, they will see the new project added to their project list and will be redirected to the permissions index page. Because the newly invited user is a manager, they can only view permissions, not add additional users.

![](https://i.imgur.com/nIyeeY3.png)


### Permissions

The majority of the permissioning functionality is encapsulated in the Permission class and controller, however I also relied on application-level `before_action` helper methods to control which levels of user could take what action within a project. Here is a summary of their abilities:

| Owner | Manager | Contributor|
| All actions of manager and contributor, can invite/delete users and change permission levels, full CRUD functionality for tasks| Can perform CRU actions with tasks and sections, can assign tasks | Can complete tasks |

![](https://i.imgur.com/ikj1FqL.png)

These helper methods are called throughout the controllers for Projects, Sections, Permissions, Tasks, and Assignments, ensuring that certain functions are restricted to manager- or owner-level users.







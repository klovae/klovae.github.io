---
layout: post
title:  "JS and Rails Project Debrief"
date:   2021-05-29 13:51:07 -0500
---

## The Creation of Dinner Winner

One of the toughest tasks for me to get through each week is meal planning. The longer the pandemic has gone on, the harder it is to put the effort into finding and trying new recipes and figuring out what to eat that week. I know there are already apps for meal planning out there, but I wanted to take the opportunity to make something lightweight for my JS+Rails project.

Historically with my projects I have struggled to keep the scope more in the realm of an MVP, so I was very dedicated to making sure that this project stuck to the requirements. 

# USER STORIES

Below are my user stories:

* As a user, I can create meal a meal plan for the week with associated meals that have linked recipes and notes
* As a user, I can add tags to my meals to indicate the basic information about the meals in my plan
* As a user, I can add more meals to an existing plan
* As a user, I can delete meals from an existing plan

Here is what the basic layout looks like for a plan:

![](https://i.imgur.com/hovgAcY.png)

To make changes to a plan, a user can click the pencil next to the plan name to enable the editing functionality: 

![](https://i.imgur.com/fBap42A.png)

All of this is accomplished through DOM manipulation.
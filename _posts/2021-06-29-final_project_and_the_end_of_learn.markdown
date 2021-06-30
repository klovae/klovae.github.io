---
layout: post
title:  "Final Project and the End Of Learn"
date:   2021-06-29 11:32:40 -0400
---

# Revisiting BuJo 2.0

In the interest of time, I made the decision to revisit the app I created for my [Sinatra Project](http://elyseklova.com/2017/02/16/making_a_sinatra_app_bujo_2_0.html), BuJo 2.0. 

Bullet Journaling has only continued to grow in popularity since my project, which I completed over 4 years ago at this point. Between lavish instagram spreads and colorful, intricate layouts on Pinterest, Bullet Journaling has gotten something of a reputation for being intimidating, difficult, and/or too much work. The application I've built aims to take things back to the roots. Per my previous post, the core of Bullet Journaling is a to-do list: 

> Every day, you write your tasks down in a list with a bullet for each task. This is your "daily log". When you finish a task, you turn that task's bullet into an "X". If you can't get to it, you do what's called "migrating" -- you move the task to the next day by turning that original bullet into an arrow (">") and re-write the task in tomorrow's log.

As it is now, my BuJo app primarily concerns itself with the creation and completion of tasks, but is easily extensible to handle migration, content editing, and deletion.

![](https://i.imgur.com/CFA2H2Z.png)

## Lessons Learned

*1. React is a front-end framework; let it be concerned with front-end matters. It doesn't need to mirror or even relate directly to the setup of the API*

This was maybe the biggest one for me, moving from Flatiron's lessons and labs into creating my own application. Coming from Ruby and especially Rails, which is very opinionated about how everything is laid out and connected, JS and especially React are much more open-ended. As a result, I spend a significant amount of my first few days working on this application dealing with analysis paralysis about "the right way" to set up my front end. When does something need to be a separate component? Which components need to be connected to the store? While there are guidelines, there are no hard and fast rules and it's okay to experiment.

*2. Refactoring/recreating an existing project is not the time saver it looks like it's going to be*

As a legacy self-paced student, I am coming up on my final deadline to finish the program. With a shorter time frame than I was comfortable with, I decided to go with what I knew and create a React application based on my Sinatra project. While I won't say this was a unilaterally poor decision, I did NOT get the time savings I was banking on from adapting an existing project. All applications are gradually built up from a foundation of basics and I did not fully appreciate the amount of work that went into making my Sinatra (and later, Rails) application work so smoothly.

*3. React Router is more complex than it appears*

While basic routing, `Links` and `NavLinks` are fairly easy to implement, creating client-side routing that properly reflects RESTful conventions turned out to be trickier than I expected. Specifically I ran into two issues:

(1) DataType conflicts when locating the right props to send to my child components, and (2) timing issues when working with async actions that enable locating and sending the correct props for my child components.

Here was the heart of the Struggle:

```
class AllDaysContainer extends Component {
  
  componentDidMount() {
    this.props.fetchDays()
  }

  render() {
    return (
      <div>
        <Route path={`${this.props.match.url}/:id`} 
        render={routerProps => {     
          if (this.props.days.length < 2) {
              return <h3>Loading day...</h3>
          } else {
          return (
            <DayTasksContainer 
              day={this.props.days.find(
                (day) => day.id === +routerProps.match.params.id)}
              />)
              }
            } 
        }/>
        <Days days={this.props.days} />
      </div>
    )
  }
}
```

My `AllDaysContainer` is responsible for rendering a master/detail view of my days and the tasks that belong to a second day. Issue (1) cropped up in finding the correct Day to render in `DayTasksContainer`. You'll notice a `+` before finding the `params.id` in `routerProps`: `<DayTasksContainer day={this.props.days.find((day) => day.id === +routerProps.match.params.id)}/>`

This is because my `day.id` was saved in state as an integer, while the `params.id` coming from routerProps was a string, and with strict comparison `===`, my `find()` function wasn't finding anything. The `+` converts `params.id` into an integer to match `day.id`. `parseInt()` would also work for this function.

The second issue cropped up when using my actions and reducer to modify tasks in this master/detail view -- while my `routerProps` was working properly in this case, JS was attempting to render `DayTasksContainer` *before* `this.props.days` was properly populated. The solution was an `if/else` statement, ensuring that `DayTasksContainer` isn't returned until there are at least two days in props.

## Concluding ##

This is my last project for Flatiron School, and I cannot express fully how much this experience has taught me. If anything it is the start of a longer journey more than the end of anything I'm already looking forward to further learning with JS and React, and using my knowledge to create more and better tools for my company. It's been a wild ride so far and I couldn't be more grateful to my classmates and instructors for helping me get to this point.

To future endeavors,

Elyse


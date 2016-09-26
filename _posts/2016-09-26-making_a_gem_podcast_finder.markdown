---
layout: post
title:  "Making a Gem: Podcast Finder"
date:   2016-09-25 23:46:20 -0400
---

 It's kind of awesome to be writing this blog post. I have to remind myself that even just a couple of months ago I was reading other Learn students' posts about making their CLI gem projects and thinking "holy shit they know so much." With still so many lessons to go, I have trouble keeping in perspective how much I've learned already, so I want to use this as an opportunity to say: *hell yeah, me.*

That's not at all to say that this project went quickly or smoothly. I probably learned as much about myself as I did about coding over the past month. According to Github, my first commit was September first, and the last commit I made that I would consider "final" was this past Friday, September 23rd, just over three weeks later. In this post I want to talk about the process of putting the gem together and lessons I learned along the way.

**CONCEPT**

I talked a little bit about this [in a previous post on starting panic-free projects](http://klovae.github.io/2016/09/08/starting_a_project_data_first/), but the idea for this project came out of my recent need for something to fill the time while driving in and out of in-town Houston. When I started listening to [Invisibilia](http://www.npr.org/podcasts/510307/invisibilia) on the [NPR One](http://one.npr.org/) app, I realized that NPR and its affiliate stations regularly produce a ton of interesting podcasts that I'd never even heard of. I wanted a way to find and listen to some of these lesser-known podcasts instead of just hoping that NPR's app would find them for me.

The goal: create a CLI gem to help me quickly browse and get details on NPR's vast podcast collection.

Once I had an idea I was excited about, I went through [NPR's Podcast Directory](http://www.npr.org/podcasts) more thoroughly, thinking through what data was available and how it would translate into classes and instances once I started programming. One of the things I did that was super helpful was diagram out each class and the data I would need to make instances of each class:

![](http://i.imgur.com/rDg31hw.jpg)

This helped me think through what data I would need to scrape and how everything needed to fit together. You can also see some dreaming about possible additional data to gather, like going to Wikipedia to get NPR's station list and add locations to my Station instances.*

*At the start of the project, my CLI was going to have options for browsing podcasts by category, by alphabetical order, and station. Toward the end of week 2 I decided to limit the scope of the project for the sake of finishing the project before 2018. I'll talk more about  that below when I go over lessons learned.

**SETTING UP MY FILES**

Once I had a good idea of what I wanted to start scraping, I started the directory in Learn's IDE, did my official git init and got my remote repo set up.

![](http://i.imgur.com/z6uhBuc.png)

In hindsight, the most important thing that I should have done (but didn't) was [use Bundler to create the gem structure](http://bundler.io/rubygems.html).

To be honest, I straight up avoided the gem part of this project until my program was essentially done and I couldn't put off packaging it any longer. At the time I told myself it was because I was super excited to get to work on the program part, but looking at it from this side of being done, it was really just pride: I knew I could set up the directory, gemfile, and environment just fine on my own and didn't want to rely on Bundler to make me one.

But just because you know how to do something doesn't mean that you should. What I would realize later is that getting your up on Rubygems.org and making sure that it installs and runs on other people's computers is a lot less straightforward than it seems. There's a reason the command `bundle gem name_here` exists -- it takes a lot of the guesswork out of the process and lets you focus on the programming instead of the packaging.

**GETTING MY DATA: EXPERIMENTS IN SCRAPING WITH NOKOGIRI**

I'm keeping this section short because [I've already written about the issue that came up]( http://klovae.github.io/2016/09/08/starting_a_project_data_first/) when I started trying to write my scraper methods and [put together a tutorial on the way I eventually solved it](http://klovae.github.io/2016/09/23/tutorial_how_to_scrape_infinite_scroll_websites_with_nokogiri/).

The TL;DR version is that NPR's Podcast Directory uses an infinite scroll script to provide users with more content as they scroll down on a given page. What that meant for my project was that Nokogiri wasn't able to scrape the page as-is because the content is literally not part of the page until user input (scrolling) tells the page to load more. The good news is that infinite scroll scripts have to load their content in from somewhere, and once you find the URL for the source of the content, you can just scrape that URL instead.

**BUILDING OUT MY OBJECTS**
Once I was able to actually get my data from NPR's website, I started working on my classes, instances, and methods for my objects. Here's the basic structure that emerged from my diagramming session:

**Top level: Category class**, which contains all categories through an `@@all` class variable. Each `Category` instance has many `Podcast`s.

**Second(ish) level: Station class**, which contains all the station data through an `@@all` class variable.
Each Station instance has many podcasts (which the station produces), and keeps track of them through the `@podcasts` array. Stations belong to categories through their podcasts.

**Third level: Podcast class**, which contains all podcast series through an `@@all` class variable. Each `Podcast` instance belongs to a station and also may belong to one or more categories. Every `Podcast` instance has many Episodes, which it keeps track of through its `@episodes` array.

**Fourth level: Episode class**, which contains all episodes through an `@@all` class variable.
Every `Episode` instance belongs to a `Podcast`.

Having all this laid out in my diagram ended up making this portion of the project probably the easiest. I appreciate that Learn set us up pretty well for this part. I had an easy time getting my objects and their methods up and running and collaborating. Given the number of classes I was working with, I decided to create separate scraper and importer classes to make it easy to identify where errors were coming from as I worked. This was the first project I've done without written tests and spec files, but I spent a lot of time running methods and creating instances using a console to make sure everything was working together. Tests I wanted "passing" were as follows:

* Scraper methods pull in the correct data and return arrays of hashes that will be used to instantiate objects
* DataImporter methods instantiate objects using scraper data from all classes: Category, Station, Podcast, and Episode
* Objects instantiated with hashes from the scraper methods have the correct instance variables.
* Objects instantiated by DataImporter are correctly associated: Categories have many Podcasts, which in turn know what Categories they belong to. Stations have many podcasts, which in turn know what Station they belong to. Podcasts know what Episodes they have, which in turn know which Podcast they belong to.
* I can string together attr_accessors from multiple classes without raising errors or getting incorrect data. For example, I can ask the arts category the name of the station that creates the first podcast in its collection by calling .podcasts.first.station.name on it.

**THE COMMAND LINE**

After I had the basics of my objects working together and my scraper and importer methods bringing the data in, it was time to start my command line. At this point, I was setting out with a starter menu that had the following options:

1. Browse podcasts by category
2. Browse podcasts by alphabet
3. Search podcasts
4. Get a random selection of podcasts

I started with the first menu item and got to work. The user experience was a huge priority for me going into this project, because I knew with all the data available in multiple levels of information, it was going to be important to be able to go up and down levels smoothly. How much of a pain in the ass would if be if every time you wanted to look at a different podcast's episodes you had to start back at the category menu? I also wanted the user to be able to exit, see a list of commands, or get back to the main menu at any point in the program.

Stating those requirements and actually fulfilling them turned out to be very different things. In the Tic Tac Toe with AI project, my partner and I used a `while` loop to implement our users' ability to exit at the end of the program, but what I wanted to do with this program required a more complex solution. In some ways the difficulties of implementing this kind of functionality are unique to command-line programs -- once you have a visual interface like a web browser, it's as easy as using a dropdown to access a menu or clicking the X on your tab to exit the site.

What I ended up doing to make it work was creating custom methods for getting input and parsing it based on commands a user might type. I then filtered the input through another method that either took actions based on that input (like showing the menu, showing the help list, or exiting the program) or passing the input on for other methods to use.

Here's what it looks like:

```
 def get_input
    input = gets.strip
    self.parse_input(input)
  end

  def parse_input(input)
    if input.match(/^\d+$/)
      @input = input.to_i
    elsif input.upcase == "HELP" || input.upcase == "MENU" || input.upcase == "EXIT" || input.upcase == "MORE" || input.upcase == "BACK" || input.upcase == "PODCASTS"
      @input = input.upcase
    else
      @input = "STUCK"
    end
  end

def proceed_based_on_input
    case @input
    when "STUCK"
      puts "Sorry, that's not an option. Please type a command from the options above. Stuck? Type 'help'.".colorize(:light_blue)
    when "HELP"
      self.help
    when "MENU"
      self.browse_all_categories
    when "EXIT"
      @continue = "EXIT"
    when @input == "BACK" || @input == "MORE" || @input == "PODCASTS"
      @input
    when @input == Fixnum && @input >= 1
      @input
    end
  end
```

Creating an instance variable, `@input` then made it much easier to pass that input on to other methods to take action. So at any point where input is needed, it gets passed through this kind of decision making machine. The other nice thing about this process is that I can combine it easily with other methods. For example, if a user is looking at a list of episodes and needs to choose one to get details or go back to a different level, I can use `get_input` which parses the input automatically and then write additional code to send that input through another decision making machine with custom options relevant to that level.

Here's the code for choosing an episode:

```
  def choose_episode
	  self.get_input
	  if @input.class == Fixnum && @input.between?(1, @podcast_choice.episodes.count)
	    @episode_choice = @podcast_choice.episodes[@input-1]
      self.display_episode_info
	  elsif @input.class == Fixnum && !@input.between?(1, @podcast_choice.episodes.count)
	    puts "Sorry, that's not an episode option. Please enter a number between 1 and #{@podcast_choice.episodes.count} to proceed."
	    self.choose_episode
	  elsif @input == "BACK"
	    @podcast_counter = 0
	    self.browse_category
	  else
	    if @input == "MORE"
	      @input = "STUCK"
	    end
	    self.proceed_based_on_input
	    self.choose_episode unless @continue == "EXIT"
	  end
	end
```
	
You can see here that this method gets input, and if it's a number that corresponds with the episode numbers, it will display that episode. If it's a number but doesn't correspond, the program reminds the user what numbers are valid and restarts the method to get input again. If the input is "BACK", then the program goes back up a level to look at the podcasts in the user's chosen category. For all other input, the following applies:

> If the input is "MORE", which is not an option in this situation, this method re-interprets the input as the user being stuck. From here, the input goes through the proceed_based_on_input decision maker, which will catch and act accordingly if the user inputs the standards: MENU, HELP, or EXIT. In the case of STUCK or HELP, the user will be returned to the choose_episode method and asked for input again.

In the future I'd like to go deeper into using recursion this way, and how to tell when you need it, because I really struggled with how to create code to meet my requirements, and Googling "writing recursive menus" wasn't a huge help. Granted, I'm still leveling up my Google-fu, but general research into recursion turned up a lot of CS101 kind of abstract explanations that I had trouble applying to what I was trying to do.

The good news is that with these sets of code for getting input and making choices made it possible for me to implement the user interface I wanted. It was also super exhausting, and once I got the first menu item, "Browse podcasts by category" working, I decided that it was time to revisit the scope of my project and do some trimming.

One of the requirements for the CLI gem is the following: "Data provided must go at least a level deep, generally by showing the user a list of available data and then being able to drill into a specific item."

Given that my code already met that requirement by going Categories to Podcasts to Episodes, I made the tough decision to stop there, spend the remaining time making things work well and look nice, and move on to the final step: packaging Podcast Finder as a gem.

**PACKAGING MY GEM**

And now we're back to (what should have been) the start. 

Because of my stubborn refusal to use bundle gem to create my scaffolding, I ended up spending WAY TOO MUCH TIME trying to get my gem packaged, uploaded to [Rubygems.org](http://www.rubygems.org), and running on other people's computers. Along the way I learned a lot about Gemspecs, dependencies, and ultimately knowing when to give up. 

Without going into too much detail, after trying to shoehorn my program's code into the gem packaging for a few days, it ended up being much easier to `bundle gem podcast_finder`, make a new directory, move my files into the new framework, and upload to Rubygems that way.

Of course the gem worked on the first build.

If you'd like to try it out yourself, I'd be totally delighted. Feel free to download it at Rubygems.org or just run gem install podcast_finder in your terminal and call it with podcast_finder. You can also scope out my repositories: the first, which contains all the commits for the bulk of the program, [is here](https://github.com/klovae/podcast-finder-gem), and [the second final repo is here](https://github.com/klovae/podcast_finder) and contains all the commits for the working gem.

Big thanks to to my fellow Learners on the OO-Neighbors slack channel for helping me to test everything, especially Brian Reynolds, who was super helpful in breaking my menus : )

**WHAT I LEARNED**

A list in no particular order, to be read before I start another project of similar magnitude. Feel free to crib if they're helpful:

1. I have the ability to solve any coding issue/challenge that I come up against
2. BUT just because I *can* solve an issue myself doesn't mean it's necessarily a good use of time
3. There is a difference between stubbornness and grit, and I need to check in with the people around me to help me figure out which one is at work.
4. If there's a framework to help me get started, I should use it because there's probably a reason someone created it.
5. The preparation for the project is more important than starting the project
6. Adequate preparation (i.e. reading available resources) can save a lot of time over the course of a project even if it seems like it's delaying the start.
7. "Breathe. It’s just programming - no one is dying" -Corinna Brock, Learn Instructor
8. Just because other people finish the same project in less time doesn't mean I'm failing.
9. Rather than starting with a huge scope and cutting it later, start with a smaller scope and then add.
10. Other people are a hell of a lot faster at finding bugs and interface issues than I am, and thorough testers are gold.
11. Regular self check-ins are even more necessary when things aren't going well or I'm feeling stuck. 
12. Instead of digging in and trying to brute-force a solution to the problem I am having at the time, I need to step back and asking if I'm taking the right approach or if I need to pivot.

I'm looking forward to my assessment this Tuesday morning, but until then, I'm planning to use tomorrow to relax a bit, make some art, and recharge my batteries before I start on the next part of my Learn journey: SQL!

Thanks for reading.


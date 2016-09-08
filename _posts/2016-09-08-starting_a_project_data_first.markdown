---
layout: post
title:  "Starting a Project: Data First"
date:   2016-09-08 17:08:24 -0400
---


*"The first thing to know when you're picking your research topic is that you don't have a project without the data. The data has to come first."*

That mandate came from my economics professor, Dr. Pepper (his actual name), on the first day of my senior year public policy seminar. That class was the last thing standing between me and finishing my econ degree. Every unfortunate soul in that room (myself included) had work around their senioritis to develop, research, and analyze a public policy issue using econometrics.

Dr. Pepper strongly encouraged us to tackle topics where data, especially clean data, was readily available through the US Census, tax and housing data, or any of number of large data sets compiled specifically for the kind of work we were doing. Unfortunately, even these large datasets have significant limits, and I'd come up with a number of super exciting! research topics only to have my professor tiredly shake his head and inform me that the data I would need didn't exist. *Undergraduates.*

Some students felt strongly enough about their project idea that, even after it became clear that the data was hard to find, they decided to stick with it. Over the course of the semester our class got a real-time lesson in Data First: while those of us with easily-assembled data moved on to our analysis, our other classmates struggled to find, compute, and scrape together enough data to produce legit results. A few of them even had to pivot before the end of the class because **ultimately you can't make data where there isn't any available.**

**DATA FIRST IN CODING**

I almost learned the hard way how important this principle is for coding. I'm working on my final project for Learn's Object Oriented Ruby section, a RubyGem that provides a command line interface to an external data source. Right now I'm living in Houston, pretty far out in the burbs, so I've been using NPR's NPRone app to find and listen to podcasts to do something useful during the 40+ minute drive in and out of town. NPR actually has a METRIC TON of podcasts, because in addition to NPR-produced podcasts, local NPR stations also produce their own content and it's all hosted [here in NPR's podcast directory](http://www.npr.org/podcasts/).

So I thought wow! Wouldn't it be great to use my CLI project to sort through all these gazillion podcasts? But instead of taking the time to be sure the idea was feasible, I got caught up in planning. I did a quick perusal of NPR's website and assumed that the data I needed would be scrapable with Ruby's Nokogiri gem. I came up with my requirements, got my repo set up, diagrammed out my classes, and started setting up my file structure.

To make my program work, I need to scrape NPR's podcast categories and use the category URLs to scrape the individual podcast series from each category page. Despite my initial optimism, I ran into trouble when I started building out my Scraper class:

**NOKOGIRI, MEET INFINITE SCROLL**

[If you look at a category in NPR's podcast directory](http://www.npr.org/podcasts/2000/arts), the site displays an initial set of podcasts from that category, probably the most popular. If you want to see the other podcasts in that directory, you have to manually scroll to the bottom of the page and wait for them to load. When I did my initial look-through of the site, I noticed the infinite scroll but figured it wasn't going to be an issue.

While working on my helper methods to scrape the category page, I took a closer look at the HTML and realized that the contents of the infinite scroll aren't even part of the page's HTML until you scroll to the bottom. From the standpoint of using Nokogiri, I'm only able to scrape the popular podcasts for any category. Given that one of my major motivations for this project was exploring some of NPR's less well-known podcasts, I started to worry that the project was superfluous  — I don't need a CLI gem to find out about Car Talk or Fresh Air, I literally just need to turn on the radio.

So what now? In the following order:

- Panic that the time I've spent on this project were totally wasted
- Make tea
- Get to Googling
- Realize that I'm probably dealing with Javascript and AJAX
- Debate learning all of Javascript before looking at this project again
- Mope that my other CLI gem project ideas are boring
- Get back to Google, [find an answer that gives me some hope](https://www.quora.com/How-do-I-scrape-a-page-with-infinite-scrolling-when-Im-not-sure-about-the-technique-it-uses)
- Spend more time than I care to admit scouring the HTML for NPR's Podcast Directory
- [FIND THE HOLY GRAIL](http://www.npr.org/podcasts/2026/government-organizations/partials?start=placeholder)

Every time the user scrolls to the bottom on one of the category pages, the site pulls data from a secret page buried in the HTML that contains all the podcast series listed in that category. I spent so much time looking for the secret page in the site's HTML that I was starting to doubt its existence, but it's real and totally scrapable. I'm still not 100% sure how the AJAX requests work, but I know I can modify the URL after `start=` to get all the podcasts for each category. I'll post about the technical solution tomorrow but if you're starting a similar data-driven project I want to save you from steps 1-6 if I can:

**STEPS TO START YOUR PANIC-FREE PROJECT:**

STEP 1. IDEAS
Come up with the domain you want your project to cover. If you're stuck deciding between multiple ideas, I'd suggest choosing the one that has the easiest-to-scrape data. In my case: NPR podcasts.

STEP 2. DATA NEEDS
Figure out what data you need to collect. List out the specifics if you can. I find it helpful to map out my classes/instances and their relationships. For my podcast program, I need the title of each podcast series, the radio station that produces the series, and the category for each series.

STEP 3. IS THE DATA PRESENT?
Before you do any more work and especially before you start programming, SCOUR the website that is providing your data. Use your list from step 2 and make sure you can find each piece of data you need.

STEP 4. IS THE DATA ACCESSIBLE?
Make sure the datapoints that you want to scrape are accessible. By that I mean you can actually get to CSS selectors for the elements containing your data, if you're using Nokogiri. If you're looking at pulling from a table full of generic `<tr>`s and `<td>`s, make sure you can figure out how to iterate over it to get the cells you need. If one data source is giving you trouble, see if it's available elsewhere (if applicable; depends on what data you're scraping)

If you've gone through steps 1-4 and you can confidently answer YES to steps 3 and 4, you're ready to start coding!

Next up: Using Nokogiri with Infinite Scroll

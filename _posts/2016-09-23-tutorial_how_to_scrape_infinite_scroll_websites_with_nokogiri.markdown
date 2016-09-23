---
layout: post
title:  "Tutorial: How to Scrape Infinite Scroll Websites with Nokogiri"
date:   2016-09-23 16:04:06 -0400
---


Infinite scrolling is a common website feature that lets users browse products, news stories, social media posts, and other content with a similar structure. Pretty much if you've used the internet at all in the past few years, you've encountered infinite scroll at some point. It can be a great way to present lots of content on a manageable scale for users while speeding up load times by not loading extraneous data up front.

Sites that use infinite scroll also pose a challenge for scraping because the only way you can "activate" more content is to keep scrolling down the page. [As I found out the hard way](http://klovae.github.io/2016/09/08/starting_a_project_data_first/), if you're using Nokogiri to scrape, that means by default you're only going to get the pre-scroll information that shows up on landing.

The first important thing to know is that *all is not lost* (though I was 100% it was at first).

**Successful scraping a site that uses instant scroll requires three general steps:**

1. Finding the source URL and query string that the infinite scroll script is using to get more content
2. Understanding how the query string works to load more data, and what happens when it runs out
3. Incorporating code into your scraper to *responsibly* increment the query string and stop at the end.

At a high level, what's happening on the back end is that the website you're scrolling as the user is loading in more content using a combination of JavaScript and AJAX. That said, **you don't actually need to know JavaScript, jQuery or AJAX to be able to get the data you need**. I don't (yet), and I was able to 

The important bit for our purposes is that in order to get more content, the script for the infinite scroll is using a specific URL and incrementing a query string ([which you can read more about on Wikipedia in the "structure" section](https://en.wikipedia.org/wiki/Query_string)). Once you find that URL, you can can modify the query with your code to replicate what the script is doing and get the data you need.

In some cases the content that the infinite scroll script is pulling from will be displayed in JSON or XML, but I'm going to focus on regular HTML sites for this tutorial.

Let's go through NPR's Podcast Directory as an example and see how we can save the day with some detective work and crafty use of Nokogiri. I'll also be supplementing with a look at Pop Chart Lab's website to show another common but slightly different setup and how to approach it.

**PART I: SCAVENGER HUNT**

**STEP 1: Investigate what's happening on the page.** When you start browsing one of the categories in [NPR's Podcast Directory](http://www.npr.org/podcasts/), you see this:

![](http://i.imgur.com/ClF4HYE.jpg)

Eight podcasts. Looks like that's it. But wait, what happens when you start scrolling?

![](http://i.imgur.com/2xJsyVI.jpg)

More podcasts load beneath what was previously the bottom of the page! Where are they coming from?

**STEP 2. Look under the hood**. Open up devtools and use the element selector to figure out where the action is happening. In our case, all the podcasts are contained in a section with the id `podcast-section-core`, including the new ones that load when you scroll. There's a good chance our link is somewhere in here.

Once you've identified the correct section, start exploring the surrounding HTML. You can also reload your page and watch the section to see what's happening in the code as the infinite scroll data loads.

Here's NPR's page in action:

![](http://i.imgur.com/sEqWwpR.gif)

You can see new `article`s are being added to the page's HTML automatically! As a longtime resident of the internet, seeing a feature like this in action from a student perspective is kind of awesome, I'm not gonna lie. I had no idea what was going on behind the scenes. Before scrolling, the div `<div id="infinitescrollwrap">` is empty, but as soon as new content loads, a new div appears, `id="infinitescroll"`, with a whole new set of podcasts.

**STEP 3. Spot the link.** Keep poking around the relevant sections. What you're looking for is a URL that ends in the following structure:

`/blah-blah?stuff=word` (`word` might also be a number).

Shopify sites commonly make use of infinite scroll if they have a lot of products. One of my favorite data visualization companies, Pop Chart Lab, uses Shopify for their website, and their infinite scroll link looks like this:

`https://www.popchartlab.com/collections/prints-all?page=8`. You can manually change the end number to see new "pages" of content: the same ones that would load automatically if you scrolled.

Looking back at our NPR example, where's our link?

![](http://i.imgur.com/5k09a1d.png)

THAR SHE BLOWS.

```
  <div id="scrolllink" style="display: none;">`
  <a href="http://www.npr.org/podcasts/2051/society-culture/partials?start=placeholder">More from Society &amp; Culture</a>`
  </div>
```

Handily listed under `div id="scrolllink"`. I'm not going to admit how long it took me to find this link the furst time because looking at it now, it's *right there* and it has its own ID. In my defense, I wasn't even sure what I was looking for at the time.

Here's what Pop Chart Lab's HTML looks like:

![](http://i.imgur.com/pSH3GUL.png)

In this case, the link for the infinite scroll is buried along with the rest of their products, but you can see that instead of a product ID, the ID here is `more`, which is a good clue.

**STEP 4. Familiarize yourself with the contents of the "new" site.** Follow the link to see what you'll be scraping. In the case of Pop Chart Labs, it's just a paginated version of what you see normally:

![](http://i.imgur.com/lQeANfc.png)

In the case of NPR's podcasts, though, you're looking at a super-basic, CSS-less placeholder site that doesn't have anything more than the content for each of its podcast "items". Take a look:

![](http://i.imgur.com/osTM3DB.png)

For me, this means that sorting out my selectors and coding the actual scraping part is relatively easy. The challenge turns out to be figuring out how to increment the query string.

**PART II: URL CRYPTOGRAPHY**

Here are the questions you'll want to be able to answer before proceeding to Part III:

* How to increment the query string
* How many items load with each incrementation (if it's always the same)
* What information you want to capture when you load a new page and how to get it with Nokogiri.
* What happens on the page when you run out of content? I.e. how will you teach your code to know when to stop?
* What is the maximum amount you can increment the query string while still capturing all the data once?*

*Because you'll be hitting the site you're scraping each time your code increments the query string, you want to minimize the number of times you have to do this. We're not trying to DOS-attack NPR's site; we just want to listen to some podcasts. 

Keep reading below for more detail or skip to Part III if you've got your answers and you're ready to code.

**STEP 5. Learn how to increment the URL.** If you're trying to scrape a site that uses pagination like Pop Chart Lab, this part is easy. With a URL like `https://www.popchartlab.com/collections/prints-all?page=1` all you'll need to do is increment the number by one each time to get the next page.

NPR's site proves to be a lot trickier for a couple reasons. The first is that it's not straightforwardly paginated, and the second was that, as I discovered later, not all podcasts contained in the placeholder site are actually being displayed. The URL also doesn't give much information either: `/partials?start=placeholder`. If you're having trouble figuring out your site, here's a general process to follow:

1. Query strings use a `key=value` structure, so start playing with the value. What happens if you try `start=0`? `start=1`?
2. Count how many separate "items" are being loaded each time you change the value
3. If the number of items being loaded seems different each time, check the HTML. In NPR's case, only certain podcasts were being displayed
4. Check the HTML on the original page and see if you can find hints for what's being loaded. Going back to where I originally found my scroll link, another look around reveals that `div id="infinitescroll"` had a `data-item-selector` attribute that was set to `"article.podcast-active"`. Bingo.

While it turned out that NPR's query string loads 24 podcasts by default, only the ones that are class `podcast-active` are even visible. This threw me off for a while. Just keep trying different URL increments, counting your items, and keep an eye on the HTML, not just what's showing up on the page.

**STEP 6. Learn what happens when content runs out.** This is important to telling your code when to stop scraping without causing an error party. Unfortunately I don't have a better alternative at this time than trial and error, though I may revisit this issue after I've learned some JS. For now, keep incrementing your query string until you run out of content (both displayed and `display: none`d).

![](http://i.imgur.com/er26kZy.png)

In the case of NPR's podcast directory, you're left with an empty `<div>`.

Pop Chart Lab's HTML also makes it pretty easy. Sup `<div id="product-list-foot">`!

![](http://i.imgur.com/y3grc5H.png)

**PART III: WRITE YOUR CODE**

**Step 7. Think through your pseudocode.** By now you should have a pretty good idea of the pseudocode you'd need to scrape your site. For NPR's site it would be something like:

```
counter = 1
until div is empty
scrape http://www.npr.org/podcasts/2051/society-culture/partials?start=(counter) 
  get your desired content with css selectors
	do something with that content (make a hash, save to an array, etc.)
counter += the number of podcasts we just scraped
end
```

Pop Chart Lab's scraper pseudocode would look almost the same except we're incrementing until we reach `div id="product-list-foot"` and only increasing `counter` by 1 with each loop.

**Step 8. Write and test your code.** 

The actual code is obviously going to differ project to project. For reference, here's the code I'm using to scrape podcasts:

![](http://i.imgur.com/pR4e3Xa.png)

You can see that rather than hard-coding the number of podcasts, I set the counter to increase by the number of podcasts loaded. This is what I meant above about making sure you're only hitting the site your scraping the minimum number of times you need to get the data. This counter ensures that, if for some reason NPR decided to display more podcasts per "scroll" in the future, my code would adjust accordingly.

As always, especially with Nokogiri, make sure you test your code relentlessly so you're scraping the right content, and adjust for any formatting weirdness that comes from line-breaks. You want your data to come into your program as neat and clean as possible.

If you have any questions about this process or need a hand with cracking the code (ha) on the website you're trying to scrape, feel free to reach out to me on Slack if you're with Learn or via Twitter otherwise. I'm @klovae both places.

Happy coding!


**MORE RESOURCES:**
* [Harsh Bhimjyani's Blog: Scraping from a website with infinite scrolling](https://harshbhimjyani.wordpress.com/2015/05/04/scraping-from-a-website-with-infinite-scrolling/)

* [Quora: How do I scrape a page with infinite scrolling when I'm not sure about the technique it uses?](https://harshbhimjyani.wordpress.com/2015/05/04/scraping-from-a-website-with-infinite-scrolling/)



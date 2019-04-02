---
layout: post
title:      "CLI Gem Project #1 -- Marvel Movies"
date:       2019-04-02 18:07:09 +0000
permalink:  cli_gem_project_1_--_marvel_movies
---


I had an absolute blast with my first Flatiron Project ([check it out here](https://github.com/mathycoder/marvel)).  While I learned a ton from the test-driven labs up to this point in the curriculum, the open-endedness of my first lab helped me learn a lot more about coding.  To even get started with the project, I had to learn how to set up an environment and communicate with Github.  Then I had to really think through what I wanted my project to do and how the different classes would communicate with each other.  

![](https://www.dropbox.com/s/qholc7yja5036s3/Screen%20Shot%202019-04-02%20at%201.34.55%20PM.png?raw=1)


# Refactoring Code with #send
But what really made my learning more concrete was code refactoring.  I really tried to follow the mantra of doing anything to make the program work the way I wanted, and then making the code more efficient.  This process became more and more addicting as I explored all of Ruby's tools.  

Here's an example.  My program allows you to sort the Movie instances by their various attributes--@rating, @us_canada_gross, and @worldwide_gross.  Here was my initial strategy.

![](https://www.dropbox.com/s/d6na51zqp6tvp9f/Screen%20Shot%202019-04-02%20at%201.42.38%20PM.png?raw=1)

Some of this code was really repetitive.  But I couldn't really think of a way to refactor it, since the instance variable I sorted by changed for each method, and you can't call an instance variable using a string.  But then I remembered about #send!  I realized I could call the instance variables this way, which would allow me to combine all three methods into one. 

![](https://www.dropbox.com/s/x1b0o43bt7hf39k/Screen%20Shot%202019-04-02%20at%201.53.22%20PM.png?raw=1)

Notice when I call the various 'sort by' methods, I'm using an argument with the string of the attribute I want to sort by.  I refactored #.sort_by_rating, #.sort_by_worldwide_gross, and #.sort_by_us_canada_gross all into one #.sort_by(attribute) method.  The trick was using #send.  Now I accomplish 'movie.rating' or 'movie.worldwide_gross' flexibly with 'movie.send(attribute).'  This abstracting of the code would also allow me to easily sort by other attributes as I scrape them in the future or add more functionality to my application.

Moving forward, I plan to refactor more of my labs.  I think it helps me internalize what I've learned and will help me become a better programmer!  








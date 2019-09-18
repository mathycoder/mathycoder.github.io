---
layout: post
title:      "MyGradebook with JS"
date:       2019-09-18 17:26:32 +0000
permalink:  mygradebook_with_js
---

![](https://www.dropbox.com/s/uo6jo3ep7s8vku7/Screen%20Shot%202019-09-18%20at%2011.35.14%20AM.png?raw=1)

Working with MyGradebook and incorporating interactive features was a challenging introduction to how JavaScript changes the nature web development.  As always, engaging in a project for the Flatiron School means problem solving in ways you didn't anticipate.  The deeper you go with one of your projects, the more challenges there are to solve.  Here I'd like to share some of those challenges and insights into JavaScript I had will pushing through this project.  

# Oh, so that's what rendering the entire front-end with JS would look like.  
After spending months really making sense of Ruby on Sinatra and Rails, I felt like I was getting pretty good at rendering Views using Rails helpers, custom helpers, and ERB.  Then the floor was ripped out from under me.  My technical coach shared that Single-Page Applications pretty much use a line like the one below when navigating between different pages:

```
$('main')[0].innerHTML = ''
```

I honestly was dumbstruck.  The idea that entire webpages could be rendered using DOM elements makes sense, but it just ran counter to everything I had come to understand about web programming so far.  But I love how the Flatiron School has designed these units to build off one another.  I can build a solid front-end using Rails.  Now here was another challenge: to do it just with JavaScript.  

# Connecting OO Ruby to OO-ish JS
I couldn't imagine rendering my entire class show page using JavaScript.  There are so many associations and specific methods for each Model needed to render even one row in the table.  How would I do this with JavaScript?  And where do I even put the "View"?

The first thing I tried to do was create ES6 Classes for each of my Models.  It helped me through this project and throughout this unit to make explicit connections between JS and Ruby, something I hope the Learn.co curriculum eventually does explicitly for future students.  For example:

![](https://www.dropbox.com/s/a7vszorxyoyz9kw/Screen%20Shot%202019-09-18%20at%2012.16.36%20PM.png?raw=1)

Some helpful connections:
1. Initializers and Constructors are parallels in Ruby and JS.  
2. Both self and this refer to the current instance being created.  
3. As far as I can tell, there really isn't a good way to do class variables in JS yet.  To create the parallel of @@students in JS, I created a global variable 'students' and push new instances to that array.  
4. snake_case vs camelCase!
5. The comparison of #sort_by vs .sort() is always helpful to get at the core of Ruby and JS; the former uses a block, the latter a callback function.

It can honestly be hard to keep the differences between these two languages straight.  It can also be challenging remembering earlier notation like attr_accessors and #initialize after getting used to ActiveRecord abstracting all of that away.  Making comparisons between the two throughout the lessons was immensely helpful for building and reviewing knowledge.  Converting all of my Ruby classes into JS Model Objects also helped!

# Rendering an entire Show Page
With Classes for each of the associated models needed to render my klass/show page created, it was time to make the the AJAX call.  (Blogging provides a fantasy narrative; in reality, I spent hours figuring out how to get $(document.ready) to work and solve a whole host of other issues.  But we'll keep it simple!)  

```
function getData(klassIdFromLink = undefined) {
  $('main')[0].innerHTML = ''
  const klassId = klassIdFromLink || window.location.href.split("/")[4]
  $.get(`/classes/${klassId}.json`, function(json){
    klass = new Klass(json)
    new Teacher(json.teachers[0])
    createJSONObjects(json.students, Student)
    createJSONObjects(json.assignments, Assignment)
    createJSONObjects(json.learning_targets, LearningTarget)
    createJSONObjects(json.standards, Standard)
    createJSONObjects(json.grades, Grade)
    renderShowPage()
  })
}

function createJSONObjects(json, cla){
  for (i = 0; i<json.length; i++){
    new cla(json[i])
  }
}
```

When is .getData() called?  Well, it can be called when a user navigates to a particular URL, or it can be called when a user clicks a link.  Therefore, I needed some flexibility for what endpoint to call in my $.get request.  Then I had to make JS Model Objects for the Klass, the Teacher, and arrays of these objects for Students, Assignments, LearningTargets, Standards, and Grades. For simplicity, the function .createJSONObjects() takes JSON and a Class Name as an argument, allowing a line like `new cla(json[i])` to create the particular Model we need.  

Now we have access to each of the objects, most of them in arrays like this:
![](https://www.dropbox.com/s/j6z4xnhxm691n3p/Screen%20Shot%202019-09-18%20at%2012.37.46%20PM.png?raw=1)

I'm still not completely happy with how this works.  I don't like using global variables to store these objects, and wish I could store them more easily within the `klass` we're rendering.  

# Where do we put the views?
Now that I'd made my AJAX call and all of the associations needed to render a Klass were stored neatly in JS Model Objects, I needed to try and render the view.  I look forward to working in React to see how they answer the organizational question of where these views should go.  I definitely ran into the problem of where to put all of this HTML and all of these calls to my JS Model methods.  Putting it in the same file as the AJAX calls would be extremely messy.  

Fortunately, this instructional video provided some guidance: [jQuery Office Hours](https://www.youtube.com/watch?v=oHPM0ekV7zQ).  In it, they used methods inside the Classes (well, the instructor used Prototype Functions instead of Classes, but same idea) to render the HTML.  I saw the benefits of this strategy as well as its organizational merits -- it's the Show page for a Klass, so why wouldn't the HTML be kept within that Class?  

Here's the beginning of my formatting  for the table:

```
  formatShow(){
    let html = ''
    html += `
      <div class="gradebook-wrapper">
        <table class="gradebook">
          <tbody>
            <tr>
              <th rowspan="2">
                <div class="gradebook-title">
                  <div id="gradebook-details">
                    <h2>
                      <strong>${this.name}'s Gradebook</strong><br>
                    </h2>
                    <div id="class-details">
                      <strong>Subject: </strong> ${this.subject}
                      <strong>Grade: </strong> ${this.grade}
                      <strong>Period: </strong> ${this.period}
                      <br><br>
                    </div>
                  </div>
                  <div id="book-logo">
                    <img src="/assets/open-book2.png">
                  </div>
                </div>
              </th>
              <th></th>
              ${this.learningTargetHeadersHtml()}
            </tr>
            <tr>
              <td></td>
              ${this.assignmentHeadersHtml()}
            </tr>
```

What I particularly liked about returning a string of HTML from within the Class was my ability to do things like `${this.learningTargetHeadersHTML()` right within the string.  This calls a method that iterates through all of the learning targets.  We get some efficient code by interpolating a call to that method right inside the HTML string!  

Once .formatShow() rendered a given klass, I attached it to the DOM with jQuery, and everything worked perfectly!  Not!

# Everything I took for granted...
The last suprise/challenge I encountered while refactoring my klass/show page from Ruby into JS was how many things broke along the way.  I was so excited to finally render a show page identical to my Rails View using only JS.  

The first surprise came when I tried clicking one of the buttons in my header.  
![](https://www.dropbox.com/s/lo5md59iokmfxg5/Screen%20Shot%202019-09-18%20at%2012.54.46%20PM.png?raw=1)

Clicking +A, +LT, or +S would add Assignments, Learning Targets, and Students to the wrong class.  Clicking Students would show a dropdown of students from the wrong class.  Of course, none of these problems would happen when rendering the View using Rails.  However, instead of rerendering the entire header from scratch each time the user navigates to a new class, I instead used jQuery to update the links and dropdown <option> elements.  From the user's perspective, this keeps the header on the screen while loading a new klass.  

The other big surprise was whenever I tried hitting refresh or pressing back.  It was like the browser had no idea where I was or had been.  I eventually fixed this problem with `history.pushState(null, null, `http://localhost:3000/classes/${klass.id}`)`.  This updated the URL in the browser without the refresh, allowing the user to refresh a given page.  It doesn't however solve the problem of navigating backwards in the browser.
# In summary
I definitely learn best when tackling tricky challenges rather than reading lessons.  Trying to turn at least part of my page into a SPA raised tons of issues and helped me begin to see how pages rendered fully by JS differ from pages rendered in Rails.  It taught me how to use a Rails backend to render JS Model Objects in the front-end.  It further highlighted the similarities between JS and Ruby, especially regarding Object Orientation.  And finally, it has heightened my anticipation for a front-end library like React.  I imagine it'll be like being introduced to Rails after working with Sinatra.  

One project to go!











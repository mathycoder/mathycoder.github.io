---
layout: post
title:      "Closure Diving"
date:       2020-02-29 12:29:21 +0000
permalink:  closure_diving
---


Recently I've been reviewing technical concepts in JavaScript to get ready for interviews.  One topic  I hear to stay up-to-date on is 'closures'.  While closures are inherent to a lot of the coding I've done in vanillaJS and React, I'm not always explicitly thinking of them or how they're working in my code.  

So this seems like a good opportunity to look through some of my code and really think through where closures are and how they're working.  

One of my favorite tests is using Chrome's dev tools to see closures within a given scope.  I can drop a `debugger` right into a scope, and check out the variables captured in closure.  So I'll definitely be using strategy along my journey.  

## But what is a closure?

There are a lot of explanations on the internet for what a closure is.  I think it was defined best in [Master the JavaScript Interview: What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36):

> "A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function."
> 

Interestingly, the term "lexical" is a reference to how the JavaScript engine compiles and runs code.  The first run through the code is the "lexical" run through, where the engine is assigning variables to memory.  So if an inner function references a variable from its outer scope, this is picked up by the JavaScript engine.  

## A 'rosier' explanation? 
[Dmitri Pavlutin: Thoughts on Frontend Development](https://dmitripavlutin.com/simple-explanation-of-javascript-closures/ ) offers a cool analogy for thinking about closures.  Imagine a painter sitting in front of a bouquet of roses.  Consider real life to the be the outer scope, and the painting itself to be the inner scope of the function.  The second the painting is done, a closure happens.  The painting has references to real-life roses, just as the outer scope has references to outer scope variables.  

My favorite part of the analogy: if you walk around with that painting, you still have your references to the variables.  A painting closes over reality, capturing objects; a closure closes over a lexical environment, capturing variable references.  

## Callback function closures 
Cool, so now with some definitions for what we're looking for, let's go closure diving!  

<img src="https://media.giphy.com/media/EPFv8HjeiYfNC/giphy.gif" />

In React, we often will render our state using a `.map()` function, which takes a callback function.  In my Student Agendas app, I use such a map function to render JSX for each of a teacher's classes.  That map function takes a callback function, which here is anonymous and defined as an arrow function.

<img src="https://www.dropbox.com/s/gl17uerezaigv54/blog1.png?raw=1" width="500px" />

Where's the closure?  Well, that callback function should have access to any variable in its lexical scope.  So if we scale outside of it, `klasses`, `editForm`, `klId`, and `klassForm` are all within the lexical scope.  

Notice I stuck a `debugger` in there.  Let's use Chrome dev tools to check on which variables were captured in the closure:

<img src="https://www.dropbox.com/s/fc0cz67otikljrw/blog2.png?raw=1" width="350px" />

Cool!  To pair with Dmitri's analogy, it's like we've painted a picture, and we can interact with those variables in the picture.  I got a bit confused why `KlassForm` wasn't included in our list of closures, but then I went through the entirety of my `.map()` function and saw that we never actually reference that variable here.  If the variable isn't referenced inside the closure, then we in effect lose it.  This makes sense; if we're painting a picture of roses, but fail to draw a bee buzzing by, we won't have the visual reference to it either.  

## Event handler closures

While the callback closure was pretty clear, I'd love to find one that completes that analogy of being able to carry a painting around and still have references to the objects inside of it.  

Below is a render function that contains the start of a form.  That form has an event listener 'onSubmit', which has another callback function to execute whenever a teacher submits the form.  

<img src="https://www.dropbox.com/s/qg3fx3i976b65b0/blog3.png?raw=1" width="700px" />

I like this one because of how event listeners work.  When this React component renders, it's going to return all of this JSX, including the form.  The onSubmit listener will initially be invoked.  That listener is returning an anonymous function, which itself has yet to be invoked.  Yet that anonymous function captures all of the variables it references outside of its given scope, or all the variables within its lexical environment.  

So let's through in a debugger to that anonymous function and see which variables have been captured within the closure:

<img src="https://www.dropbox.com/s/k8fbo3o22k35663/blog4.png?raw=1" width="350px" />


Yep!  Each variable we referenced gets captured in the closure, including any functions we reference.  This is a nice case of functions being first-class citizens in JavaScript, in that they are as versatile as all other data types. 

Does this closure better fit the analogy of carrying a painting around while still having access to the objects within it?  I think so.  That's because the closure function is attached to a form rendered to the DOM.  So no matter what changes happen on the DOM, the second we click submit this function will still be connected to each of these variables and functions.  I think that's pretty cool!

<img src="https://media.giphy.com/media/9kcBLYxWztmmc/giphy.gif" width="400px" />

## Conclusion
I can definitely say that this process of "Closure Diving", AKA looking through your code and thinking through any closures, helped clarify the concept to me.  The process of dropping a `debugger` into my inner functions too helped clarify things even further.

There's definitely more diving to be done!  I'd love to take a look at React Hooks and think through closures there.  As always, happy coding!




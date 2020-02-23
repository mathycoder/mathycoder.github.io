---
layout: post
title:      "React and Functional Programming"
date:       2020-02-23 21:21:31 +0000
permalink:  react_and_functional_programming
---


Sometimes when learning something new, it's best to be thrown in headfirst and experiment, rather than think too much about the theory in advance.  I feel like this is how the Flatiron School taught me React and Redux.  I definitely appreciate this approach; sometimes I'm much more ready to make sense of the theory after tinkering and exploring, whether it be with a new coding framework or even an open-ended math problem.  But now that I've built a few applications with React and Redux, I want to flesh out my understanding of the underlying coding paradigm inherent to React.  

At first glance, I thought React was another object-oriented programming paradigm.  After all, class components are defined using classes, and these components house state, which feels similar to instances of a class having their own instance variables.  And you can create class methods on a class component.  

But as I've been reading more about JavaScript in general, I've come to think that React aligns much more closely with a functional programming paradigm.  Here's how [Master the JavaScript Interview: What is Functional Programming? ](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0) defines functional programming: 

> "Functional programming (often abbreviated FP) is the process of building software by composing pure functions, avoiding shared state, mutable data, and side-effects. Functional programming is declarative rather than imperative, and application state flows through pure functions. Contrast with object oriented programming, where application state is usually shared and colocated with methods in objects."
> 

Let's break this down, and then draw some connections to React:
* pure functions
* avoiding shared state
* avoiding mutable data and side effects
* declarative over imperative

## Pure functions
A pure function is aptly named pure for two reasons:
1. Given an input, you will always get the same output (like a mathematical function!)
2. No side effects

Pure functions show up all over React.  The clearest example of functional programming for me shows up in Redux reducers.  Here's part of my `klassesReducer` in my Student Agendas application:

```
function allKlasses(state = [], action) {

  switch(action.type) {
    case 'CLEAR_CURRENT_USER':
      return []

    case 'ADD_KLASSES':
      return [
        ...action.klasses.map(klass => `klass${klass.id}`)
      ]

    case 'ADD_KLASS':
      const klassId = `klass${action.klass.id}`
      return [...state, klassId]

    case 'REMOVE_KLASS':
      const deleteKlassId = `klass${action.klassId}`
      return state.filter(klId => klId !== deleteKlassId)

    default:
      return state;
  }
}
```

Reducers are perfect examples of pure functions and functional programming principles.  For one, they never make modifications to the state object (here in the form of an array).  Notice that each case statement returns a modified copy of the state array.  'ADD_KLASSES' uses the non-mutating map function and the spread operator to return a brand new array.  This fits the definition of a pure function: no side effects (the state itself isn't changed or affected) and 100% predictable return values.  We'll come back to why this is awesome in a minute.

## Avoiding Shared State
A Redux reducer makes modifications to a store, or a single data object for the entire application.  Another way of putting this: we're avoiding shared state.  Contrast this to applications I've made in Python or Ruby.  I have one video game application called Pet Rescue!, where my niece Mikayla is side scrolling her way through a world trying to save all of her pets.  In this object-oriented world, Mikayla herself is an object with instance variables and methods that modify those values.  A given level is itself also an object, and tracks where on the screen we are and where the "camera" should be.  Mikayla and Level share data fluidly, or share their state.  

Functional programming discourages this, and it's interesting to think about why.  If you go back to my `klassesReducer` above, using a web application will produce a series of actions that you can observe while debugging:

<img style="margin: 0 auto; display:block" src='https://www.dropbox.com/s/5z9rrg5pzz1zt7d/actions.png?raw=1' width="500px" />

Now we start to see the benefit of using pure functions in React/Redux, specifically for making changes to state.  We can "time travel" through each action, debugging freely.  If there's a change we weren't expecting, we can go back to our list, think through the changes, and more easily spot the error than if we weren't using pure functions that caused side effects.  And since we are avoiding shared state, it keeps things simple while debugging.  

When I came across errors in my side-scrolling object-oriented world, I literally needed a piece of paper and copious notes to follow where my bug may have occurred.  Functional programming avoids shared state.  As my applications have grown in size the more I use React, I really appreciate the ease in which debugging can happen with a centralized state structure and pure functions.  

## Avoiding Mutable Data and Side Effects 
This wonderful timeline of dispatch actions provided to us by Redux would be useless were we not avoiding mutable data like the plague.  As I mentioned when discussing pure functions, once a state object is created, it is never mutated.  Instead, we make copies with slight modifications, tied directly to the actions.  So when we have a given state, and then we dispatch the action 'ADD_KLASS', we'll get a totally new object with a new property for the new klass.  Again, this allows for debugging, and prevents bugs from spiraling out of control.  

Before you draw the conclusion that the only thing functional about React is reducers, let's look at a another place where React follows this principle of functional programming.  How about the newer React hook, `useState`?  

`const [symbol, setSymbol] = useState('')`

When we use this hook, we create a state variable for a given component.  Interestingly, we never modify the variable directly.  Instead, we use a setter method provided to us:

`onChange={e => setSymbol(e.target.value.toUpperCase())}`

Setting the state in this way is interesting.  For one, it's asynchronous.  Also, it feels a lot like a mini version of a reducer.  While we're not creating an entirely new object, we are creating a new value for--in this case--a string.  So it would make sense that setState would keep some sort of history, so we could see each value we've set our state variable equal to.  Turns out, it does!  [4 Examples of useState](https://daveceddia.com/usestate-hook-examples/) talks about a behind-the-scenes object connected to the component that has "state cells", allowing you to access the previous state value.  

## Declative over imperative (procedural) programming
We're moving right along in exploring some of the ways React/Redux embraces a functional programming paradigm.  We've got pure functions, centralized state, and little to no mutability.  Perhaps you could argue that React still leans toward object-oriented programming with its class components.  I hear you, but I raise you this: React is declarative!  Most object-oriented programming languages are imperative or procedural. 

When I first started learning JavaScript, my first project was built using JavaScript Model Objects, which are basically ES6 class instances.  If I wanted to modify something on the screen, I needed to create a very procedural method within one of the classes.  Find the main element in the DOM; clear it; create a new element; attach it to the DOM.  It was kind of amusing actually.  I had this awesome gradebook app for Rails, and spent a few weeks converting it into vanillaJS.  When I was done, the application didn't really run much faster than it did in Rails.  I was surprised: everyone always talked about how much faster JavaScript was!

React in contrast to this imperative apporach is declarative.  In my Student Agenda application, when you click the button 'Edit Students', nothing procedural happens at all.  Instead, I have a declarative line of code:

```
  { editingStudents ? <EditStudents klass={klass} /> : this.renderStudents()) }
```

A state variable, `editingStudents` gets updated to `true` when a button is clicked.  This changes the value of this ternary expression, which will now return the component `<EditStudents />`.  So now the return statement of this component returns something different.  Behind the scenes, React will see what nodes in the DOM need to be added or modified.  All we do as developers is *declare* what should be on the screen given certain conditions.  

## Conclusion
It makes sense that React follows more of a functional programming paradigm than an object-oriented one.  In object-oriented programming, state can exist in multiple places within given object instances, like it did with both Mikayla and Level in my side-scrolling game.  React with Redux centralizes state in one location.

The data contained in a Redux store is essentially frozen (if you stick to the correct functional programming principles).  Each reducer action returns a new object to the Redux store, which creates a helpful timeline of states for debugging.  Other cases of state in React, like variables created by `useState`, are also have a history, and are never modified directly.

Programming in React is declarative over imperative.  Declarative code describes what something should look like: when `editingStudents` is true, the DOM knows what to render.  Imperative code would look more procedural (clear the DOM, create the new node, attach it).  

After writing this blog post, I've realized I have a bias toward functional programming.  I definitely didn't set out to write it that way.  But it just feels like functional programming developed out of years and years of developers struggling with object-oriented programming.  Each principle feels like one born out frustration--let's just centralize the state!  Let's make these functions pure!  Why can't we just declare what the DOM should look like?  

<iframe style="display:block; margin:0 auto" src="https://giphy.com/embed/fcEQaKl4KTtrq" width="480" height="233" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/the-office-michael-scott-steve-carrell-fcEQaKl4KTtrq">via GIPHY</a></p>

It makes me wonder how functional programming may evolve.  And being an absolute noob to the field, I know an exploration of other frameworks and languages will shed further light on the benefits of object-oriented programming.  

Thanks for reading, and happy coding!












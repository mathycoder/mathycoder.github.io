---
layout: post
title:      "Enter React Hooks!"
date:       2020-02-16 23:27:01 +0000
permalink:  enter_react_hooks
---

Last week I explored the Udemy course [The Complete React Native + Hooks Course [2020 Edition]](https://www.udemy.com/course/the-complete-react-native-and-redux-course/).  While my main goal with the course was to explore React Native for creating iOS and Android applications, a nice side effect was my first exploration of React hooks.  As we used hooks in the course, I kept wanting to just stick to my class components, lifecycle methods, and local state.  But eventually hooks won me over.  I still feel very new to them, but felt confident enough to solve a recent coding challenge for an employer using React/Redux with hooks.  So let's solidify these new learnings, and explore React hooks!

## Class and Functional Components
React is a framework based on components.  A component is essentially a JavaScript function that renders some JSX to the screen.  You can string tons of components together in a large component tree to manage the front-end display of an entire application.  

Components can be further separated into class and functional components.  Class components use the ES6 class notation, bringing an object-oriented approach to component design.  They are somewhat bulky, both in terms of boilerplate code and performance.  Functional components are leaner and written with function syntax, but have limited abilities.  

The added features of class components are key to what makes React awesome.  Classes in object-oriented programming languages have instance variables; a React class component has state.  As these local state variables change, React automatically triggers re-rendering to the DOM.  Class components also have lifecycle methods like `componentDidMount()` and `componentDidUpdate()`.  I would use the former in my previous applications to start an initial API call or load data into the local state the second a component was rendered to the screen.

For years, the only way to gain access to local state management and component lifecycle methods was through class components.  And so developers would give up some level of leanness for the benefit of functionality.  Meanwhile, functional components were relegated to mere display components, since that was pretty much all they could do.  UNTIL...

## React Hooks
In 2018, Hooks were introduced to React.  ([Intro to Hooks](https://reactjs.org/docs/hooks-intro.html).)  Essentially, they're a way to inject or hook state management and lifecycle methods into functional components.  They allow sleek functional components to also take on the advanced features of class components.  They also seem like a general push away from object-oriented programming to functional programming.  Let's take a look at all of the hooks React has to offer.  

## `useState` 
For my recent coding challenge, I had to create a stock market app.  One form I had to create let the user type in a stock symbol and a quanity of shares to purchase.  It was here I first used the React hook `useState()` for local state management.  Here's a portion of that component's code:

```
import React, { useState } from 'react'

const NewTransaction = ({ addTransaction, currentUser }) => {
  const [symbol, setSymbol] = useState('')
  const [shares, setShares] = useState('')

  const handleSubmit = (e) => {
    e.preventDefault()
    addTransaction(symbol, shares)
  }

  return (
    <div className="new-transaction-wrapper">
      <form
        className="transaction-form signup-form"
        onSubmit={(e) => handleSubmit(e)}
      >
        <input
          value={symbol}
          onChange={e => setSymbol(e.target.value.toUpperCase())}
          type="text"
          placeholder="Ticker"
        />
        <input
          value={shares}
          onChange={e => setShares(e.target.value)}
          type="text"
          placeholder="Qty"
        />
        ...
```

So what's going on here?  First, we import the React `useState()` with the import statement `import React, { useState } from 'react'`.  Then, we want to declare state variables separately using this function:

`const [symbol, setSymbol] = useState('')`

This is a fun use of JavaScript ES6 syntax called destructured assignment.  The function `useState()` will return an array with two elements.  The first element will be our new state object, which I named `symbol`.  In keeping with React's functional programming paradigm, we never directly change this variable.  Instead, we use a setter function, which is the second element in the returned array.  I've named this function `setSymbol`.  The argument I sent into `useState('')` was an empty string, and that serves as the default value.  

Now we have essentially our getter and setter methods for a state variable called `symbol`.  Notice how I used them within my `<input />` elements:

```
<input
     value={symbol}
     onChange={e => setSymbol(e.target.value.toUpperCase())}
     type="text"
     placeholder="Ticker"
/>
```

`symbol` is available anywhere within our component now, so we can set it equal to the value of this text input field.  What's really different from a class component is how easily we can change state values now that we have our setter function `setSymbol`.  When the user types something in the field, the event listener `onChange` triggers the anonymous callback function.  In one line, I'm able to set a new value for `symbol` using the line `setSymbol(e.target.value.toUpperCase())`.  

As you can see, it really doesn't take much to add state management capacity to a functional component with `useState()`.  But what about those awesome lifecycle methods afforded to class components?  What if I want to load API data into my local state the second a component mounts?  Well, for this, we need another hook...

## `useEffect`
React's way of injecting `componentDidMount` and `componentDidUpdate` capacity into functional components is with the hook `useEffect()`.  Again for my coding challenge I had a container component that needed to load all of a user's existing stocks and transactions into my Redux store.  I would normally have used `componentDidMount`, which triggered the actions immediately after mounting a component.  

Here's `useEffect()` performing the same action:

```
import React, { useEffect } from 'react' 

const PortfolioRouter = ({ fetchTransactions, fetchStocks }) => {
  useEffect(() => {
    fetchTransactions()
    fetchStocks()
  }, [])
	```
	
So what's happening here?  Well, just like with `useState`, we need to import `useEffect` from the React library.  Within our functional component `<PortfolioRouter />`, we can invoke the `useEffect()` function.  `useEffect` takes two arguments.  The first is the 'event' or callback function to be invoked after a render.  The second optional argument is an array that specifies when the effect should be called.  
	
For example, let's say you had a state variable called `symbol`, and you wanted some function to run every single time `symbol` changed or was updated.  Then what you would do is put `[ symbol ]` inside the second argument's array, to tell `useEffect()` to run its primary function every time `symbol` is updated.  
	
But I only want my fetch actions to run when the component "mounts."  So instead I used an empty array as my second argument.  This means that the functions `fetchTransactions()` and `fetchStocks()` will be called the first time `<PortfolioRouter />` is run or rendered.  
	
This one took some getting used to in comparison to `useState`.  But there are benefits here if you have experience using `componentDidMount()` and `componentDidUpdate()`.  One benefit the React docs cover is how often you need to include the same duplicate code inside both component update lifecycle methods.  `useState()` allows for DRYer code by doing away with the difference between mounting and updating.  If we were to exclude the second argument completely, our `effect` would run after every render and re-render of the component, including the initial one.  
	
## `useReducer`

These last three hooks are ones I haven't fully explored yet, and will probably be the subject of my next blog post.  But the Udemy course instructor did explore them somewhat, so I can give an overview.

`useReducer` and `useContext` are ways of creating a Redux-style state management structure without any external library like Redux.  

Reducers offer a different way to manage state, whether locally within a component or within the application as a whole (like in Redux).  The state itself is an object that can have many properties.  Reducers follow functional programming principles even more so than `useState`: each update to the state returns a new object AND creates a clear history of changes to the state (`INCREASE_COUNTER, DECREASE_COUNTER`...).   

Here's a reducer we used in one of our lessons, written in React Native:

```
import React, { useReducer } from 'react'
import { View, Text, StyleSheet, Button } from 'react-native'

const reducer = (state, action) => {
  switch(action.type){
    case 'INCREASE_COUNTER':
      return { ...state, counter: state.counter + action.payload }

    case 'DECREASE_COUNTER':
      return { ...state, counter: state.counter - action.payload }

    default:
      return state
  }
}
```

If you've used Redux before, you'll notice this reducer could be used verbatim within Redux.  The different really comes when using the hook within a component.  

```
const CounterScreenReducer = (props) => {
  const [state, dispatch] = useReducer(reducer, { counter: 0 })

  return (
    <View>
      <Text>Counter Screen TEST</Text>
      <Button
        style={styles.buttonStyle}
        onPress={() => dispatch({type: 'INCREASE_COUNTER', payload: 1}) }
        title="Increase"
      />
```

Notice we use `useReducer` using destructured assignment again.  Here, we are given our state variable (which we never modify directly), as well as our dispatch function.  The first argument of `useReducer()` is the reducer function itself; the second is the initial values for the state.

To make changes to the state object, we use dispatch an action object ` dispatch({type: 'INCREASE_COUNTER', payload: 1})`.  

I hadn't really made such a clear connection between state variables and reducers until exploring `useState` and `useReducer`.  They both follow a functional programming structure, keeping the actual data immutable while creating copies to take care of any changes.  A reducer really seems like the next iteration of a state variable; that clear list of previous actions allows for some helpful historical debugging.  

## `useContext` and custom hooks
One thing that makes Redux so powerful is that the store is available to any component within your application.  Using plain React would mean you'd have to send down state objects using `props`, which can quickly make your code unweildy.  I don't have much experience with `useContext`, but it basically sets up the ability to send a large state object/store to any component without having to go through all of the connecting components.  Again, if you're using Redux, you wouldn't really need `useContext`.  

Finally, I have a lot to learn about writing my own custom hooks.  The Udemy instructor used it to extract all of the state management logic out of the component itself, which is a pretty neat trick.  As I finish my coding challenge, I'll explore doing the same thing with my code.  

Thanks for reading my exploration of React hooks!  If they're new to you, I hope I've managed to intrigue you.  Happy coding!




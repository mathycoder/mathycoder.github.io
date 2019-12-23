---
layout: post
title:      "React-Redux Project Part 2"
date:       2019-12-23 16:01:51 +0000
permalink:  react-redux_project_part_2
---


![](https://www.dropbox.com/s/0vuw6n2mz86egae/login%20page.png?raw=1)

## Logging In with React

In my last blog, I went through the first challenges of my React-Redux application, Student Agendas.  I experimented with JavaScript's built-in drag-and-drop callbacks, and created a Redux store with a normalized state.  As I built out more and more of the teacher's side of the application, I realized I would need to:

1. Allow teachers to sign up and login
2. Allow students to login
3. Create routes that distinguished between teachers and students

The first challenge reminds me of a pattern I've noticed in coding: take something I've done many times in one language/framework and get it to work in another.  Logging in is smooth sailing in Rails.  We can use the BCrypt gem to encrypt our passwords and authenticate; then we let Rails know who's logged in using sessions.  

After reaching out to some technical coaches at Flatiron, one of them recommended the [Globetrotter project build series ](https://instruction.learn.co/student/video_lectures#/457) on Learn Instruct, where Howard walks us through the creation of a React-Redux app.  And it turns out, one of the options for logging in on the client side is only a slight modification of logging in with Rails on the server side!  I love when that happens!

So first, I created a controlled component called `<Login />`, which kept track of all of the form attributes like email and password.  After that, I linked the component to the Redux store and dispatched an action:

```
export function login(credentials, history){
  return (dispatch) => {
    dispatch({type: 'LOGIN_REQUEST'})
    fetch(`/login`, {
      credentials: "include",
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify(credentials)
    })
      .then(resp => resp.json())
      .then(user => {
        if (user.error){
          dispatch({ type: 'ADD_FLASH_MESSAGE', message: "Email or password incorrect" })
        } else {
          dispatch({ type: 'SET_CURRENT_USER', user })
          user.type === "teacher" ? history.push('/classes') : history.push('/myagenda')
        }
      })
      .catch(console.log)
  }
}
```

Notice the pattern here.  My actions all use the middleware Thunk, allowing for asynchronous requests.  Here, we dispatch an initial action 'LOGIN_REQUEST'.  Then we need to wait for the promise from the login fetch request to resolve before dispatching 'SET_CURRENT_USER with our received user JSON data.  I also handled flash messages here as well if the backend sends an error logging in.  

The fetch request gets us right to where we would do things in Rails: a POST request in my sessions controller.  From there, we set session[:user_id] to the ID of the teacher logging in, assuming the password authenticates.  SET_CURRENT_USER then does just what you think it would: it puts the current user data into my Redux Library (currentUser).

## One caveat: refreshing the browser

I was feeling pretty good after getting my login up and running.  Then I clicked refresh, and my application sent me tons of errors.  What happened?  Well, my Redux store no longer knew who the currentUser was.  This is kind of strange when you think about it, because on the front-end my app has no idea who is logged in, but my backend certainly does!  Refreshing a page affects the client side, but the server side still has the :user_id stored in the session hash.  

Here's where we needed an additional action.  Every time my app reloads, the first thing it does is dispatch the action SET_CURRENT_USER.  This checks with the sessions controller on the backend whether or not the sessions hash has a :current_user key.  If so, it returns that as JSON, allowing me to SET_CURRENT_USER.  

```
export function getCurrentUser(){
  return (dispatch) => {
    dispatch({ type: 'CHECKING_CURRENT_USER' })
     fetch(`/get_current_user`, {
      method: "GET",
      headers: {
        "Content-Type": "application/json"
      },
      credentials: "include"
    })
      .then(resp => resp.json())
      .then(user => {
        if (user.error){
          dispatch({ type: 'SET_CURRENT_USER_TO_NONE' })
        } else {
          dispatch({ type: 'SET_CURRENT_USER', user })
        }
      })
      .catch(console.log)
  }
}
```

So long story short, we CAN use Rails, BCrypt, and sessions to login with a React app.  We just need to initially log in with a fetch request, set the user on the front and backend, and update the frontend whenever there's a reload.  

# Private Routes
Now that teachers could log in to my application, it was time to start thinking about the difference between a student and teacher logging in.  After modifying my `<Login />` component to check for the type of login (teacher or student), and updating the backend and front-end to know whether a teacher or student is logged in, I was ready to tackle the routing side of the issue.  

Just as client-side logging in is a slight tweak on server-side logging in, the same goes for routing.  React-Router looks and acts similar to Rails Routes, but it has its own flow.  

```
          <Route exact path="/" component={Login} />
          <Route exact path="/signup" component={Signup} />
          <Route exact path="/logout" component={Logout} />
          <PrivateRoute type="student" path="/myagenda" component={AgendaContainer} />
          <PrivateRoute type="teacher" path="/profile" component={TeacherProfile} />
          <PrivateRoute type="teacher" path="/progressions" component={ProgressionsContainer} />
          <PrivateRoute type="teacher" path="/classes" component={KlassesContainer} />
```

I really like nested routing with React components.  For example, if I wanted to go to a klass show page, my route would look like `/classes/15`.  The last route above would trigger, which would render `<KlassesContainer />`.  But within that component would be MORE routes: the klasses index page, the klass show page, and deeper routes like `/klasses/:klass_id/students/:id`.  

For anyone who's used React Router, you probably notice that `<PrivateRoute />` is actually NOT a thing.  So what's happening here?  Well, `<PrivateRoute />` is actually a component that I built to look and function just like React-Router's `<Route />`.  I had help from the [Tyler McGinnis Blog](https://tylermcginnis.com/react-router-protected-routes-authentication/) blog.  

```
import React from 'react'
import { Route, Redirect } from "react-router-dom";
import { connect } from 'react-redux'

const PrivateRoute = ({ component: Component, path, currentUser, type}) => (
  <Route path={path} render={(props) => (
      currentUser !== "none" && currentUser.type === type ?
      <Component {...props} />
    : <Redirect to="/login"/>
  )} />
)

function mapStateToProps(state){
  return {
    currentUser: state.currentUser
  }
}

export default connect(mapStateToProps, null)(PrivateRoute)
```

Let's checkout this component.  One initial thing to notice is that we're just returning a React-Router `<Route />` component, which may make you wonder what's the point.  But this just keeps all of the Private Routes DRY.  This functional component ("stateless") grabs all of the props by destructuring assignment.  It sets the path, and then instead of routing to a component, we use 'render' to finess the route we want through a callback function.  

The magic happens with our ternary if-else line:  `currentUser !== "none" && currentUser.type === type ?`.  The first part is straightforward: if currentUser (which we grabbed from the Redux store) is set to "none", then this statement will be false.  This would render `<Redirect to="login" />` .  This makes sense--with no currentUser, you would need to login instead of going to this PrivateRoute.  The second part of the conditional is really fun: `currentUser.type === type`.  This is checking whether or not the currentUser's type (teacher or student) matches the type of the route.

`<PrivateRoute type="teacher" path="/classes" component={KlassesContainer} />`

So if a student tries to go to the private route "/classes", even though they're logged in, the route would redirect them elsewhere.  Only when you are both logged in and match the type required by my PrivateRoute component do you get to go to `<Component />`, which again is abstracted from whatever component you set in the props.

## Reflecting 

The nicest takeaway from `<PrivateRoute />` is how compact and useful it is.  My initial instinct would never have been to make such a small component, but it really DRYs up my code.  And it's very cool how it functions exactly the same as `<Route />`.  As I move forward coding with React, I'll look for other ways to create other compact components that will make my code DRYer and more lexical.  





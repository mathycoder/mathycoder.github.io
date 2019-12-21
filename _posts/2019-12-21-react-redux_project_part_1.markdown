---
layout: post
title:      "React-Redux Project Part 1"
date:       2019-12-21 19:22:56 +0000
permalink:  react-redux_project_part_1
---


![](https://www.dropbox.com/s/llkt01obxux62q8/Screen%20Shot%202019-12-21%20at%2012.23.10%20PM.png?raw=1)

Well, I'm finally here at the end of my Flatiron journey.  For my final project, I truly followed the guidelines set out by the project:  "It is supposed to be your magnum opus."  The app is called "Student Agendas."  My goal was to create an application that functioned both for teachers and students.  Teachers would be able to create video agendas for students.  As a teacher, I had often used videos for homework or in-class assignments, but always found it hard to individualize the videos my students needed.  This app tries to make this process easier.  It lets you create video progressions, color code them, drag and sort them into student agendas.  It captures the student side of accessing the progressions, viewing them, and submitting answers to reflection questions.  And most recently, it handles the teachers receiving submitted progressions and giving feedback through the application.  

This was a long long journey.  At the start of it, I had only done small labs involving React and Redux, both of which still felt more foreign to me than vanilla JS or Rails.  But with an ambitious project plan, I set forth on what I've learned programming and creating is all about: breaking the process into small problems that can be researched and solved, before moving on to the next problem.  

# JavaScript's Drag and Drop callbacks
Before I even began tackling Redux and making sense of a database on the frontend, I decided to work on my progression creation page.  

![](https://www.dropbox.com/s/ni9omzaa1hfi2l7/new%20progression%20creation.gif?raw=1)

I knew I wanted this page to be fun for teachers, which to me meant interactive.  So I started with my first React component, <NewProgressionContainer />.  I spent some time integrating with YouTube and Vimeo APIs to query videos, and stored these video objects in the component's local state.  Over time, I pulled most of the local state out into Redux, which made the component a lot cleaner.  More on Redux later!  But to make the page truly interactive, I started researching ways to implement drag and drop.  

When dragging a video from the YouTube/Vimeo query, I used built-in JavaScript strategies.  [MDN DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)

```<div draggable
            onDragStart={event => handleDragStart(event, video)}```

Each of my videos was wrapped in a div labeled 'draggable', which is all you need to start dragging all of your divs around.  JavaScript comes ready with the callback onDragStart, which was how I was able to keep track of which video was being dragged.  You can then store the data within an event's dataTransfer object, which is available for all drag events.  

``` handleDragStart = (event, video) => {
    let data = JSON.stringify(video)
    event.dataTransfer.setData("video", data)
  } ``` 
	
	On the agenda itself, I used the callbacks onDragOver, onDragLeave, and onDrop.  onDrop was how I accessed the video that was dropped, and then updated the state to add that video to my progression.  My event handler looked like this:
	
	```
	  handleOnDrop = (event) => {
      let video = event.dataTransfer.getData("video")
      video = JSON.parse(video)
      this.addToProgression(event, video) 
	}
	```
	
	The other callbacks--onDragOver and onDragLeave--allowed for changes to CSS when dragging a video over the progression container.  
	
# 	react-beautiful-dnd 
After I was able to drag video files into my new progression, I knew I wanted an interactive way for teachers to reorder them.  Having seen websites where you can drag an item and watch all of the other adjust, I knew I wanted that here!  After experimenting with coding this from scratch, I realized this would be a good opportunity to integrate my application with a drag and drop package.  I selected react-beautiful-dnd, mainly because it included helpful video lessons that would help me utilize their package: [Egghead react-beautiful-dnd](https://egghead.io/lessons/react-course-introduction-beautiful-and-accessible-drag-and-drop-with-react-beautiful-dnd)

![](https://www.dropbox.com/s/jemuwhu59n0t5e1/DnD%20diagram.png?raw=1)

react-beautiful-dnd works by utilizing three components, a <DragDropContext /> (the area in which dragging and dropping applies), a <Droppable />  (where things can be dropped),  and a <Draggable /> (components or divs that can be dragged).    

One new thing I learned about React along the way was the 'render props pattern,' which took me a few tries to make sense of.  Here's an example:

```
<Droppable droppableId="droppable-1" direction="horizontal">
          {(provided) => (
            <NewProgression
              placeholder={provided.placeholder}
              color={this.state.color}
              innerRef={provided.innerRef}
              {...provided.droppableProps}
              removeFromProgression={this.removeFromProgression}
              currProgression={this.state.currProgression}
              handleProgressionItemClick={this.handleProgressionItemClick}
              handleDragOver={this.handleDragOver}
              handleDragLeave={this.handleDragLeave}
              handleOnDrop={this.handleOnDrop} >
                {provided.placeholder}
            </NewProgression>
          )}
</Droppable>
```

What's happening here?  You'd think between <Droppable></Droppable> we would just put the <NewProgression /> component, since that would label <NewProgression /> as the droppable area for my videos.  But react-beautiful-dnd doesn't want to create unnecessary DOM nodes, and it has some things it needs to add to my <NewProgression/> component.  The solution is a render props pattern.  The direct child of <Droppable> isn't an additional node, but instead a callback function.  It returns <NewProgression />, so that is technically the next DOM element we'll see.  Remember, react-beautiful-dnd needs to add some props to <NewProgression />.  Returning a function allows it to render <NewProgression /> with the additional props it needs, specifically `{...provided.droppableProps}.` 

I see a similarity between the render props pattern and the Redux function connect(), which exports of a version of the Component you're making, but with state and dispatch mapped into the props as well.  It was helpful to see this in a different context to help me understand mapStateToProps / mapDispatchToProps a bit better.

# Normalized State with Redux 
I had a ton of fun getting this new progression page to look great with drag-and-drop and video APIs.  But once I was past it, it was time to really make sense of Redux.  The curriculum makes a reference to using Redux the same way we would use a database.  The idea of organizing all of my different database tables into a Redux version on the front end was truly daunting.  But as anyone who has worked with Redux for even a short while knows, nesting deeper and deeper into the store object is truly a mess to maintain.  So I was ready to create a front-end database structure with Redux.

Again, online docs made all the difference.  The most helpful one was Redux's [Normalizing State Shape](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape/).  There was one picture that made everything click for me:

![](https://www.dropbox.com/s/p37ozmsma8voxpt/normalized%20state%20diagram.png?raw=1)

So basically, each database table gets a key in the store object/hash.  But instead of then pointing at an object of all of the values, they do one more split into two keys, byId and allIds.  This seemed tedious at first, but proved really useful when rendering store data in React Components.  So much of rendering data from state involves JavaScript's #map iterator, which only works over an array.  For example, in one component I want to render all of a teacher's klasses (avoids keyword 'class').  After grabbing the klass "table" from the state, I can do this:

```
<div className="klass-rows">
        {klasses.allIds.map((klassId, index) => {
          const klass = klasses.byId[klassId]
					...
```

So I start by mapping over `klasses.allIds`, which thanks to Normalized State Shape is an array that looks like this: ['klass5', 'klass13', 'klass20'].  Within the map callback function, I use the parameter 'klassId' to then cue up `klasses.byId[klassId`.  Since `klasses.byId` contains an object of all the klasses data, this will give me the klass data I want!  

Okay, so that's the benefit of using a Normalized State Shape.  But what's the best way to create it?  
# combineReducers
combineReducers is a helpful function provided by 'redux'.  It allows many mini reducers to combine together to create your total store, which starts to feel a lot like a database.  My Redux store on the top level looks like this:

```
const rootReducer = combineReducers({
  klasses: klassReducer,
  students: studentReducer,
  progressions: progressionReducer,
  videos: videoReducer,
  reflections: reflectionReducer,
  videoSearch: videoSearchReducer,
  studentProgressions: studentProgressionReducer,
  currentUser: currentUserReducer,
  flash: flashReducer
})
```

Each database table gets its own 'table' within the store.  (The Redux docs suggest separating the true database tables from other keys like 'flash' or 'currentUser', but I kept them all at the top level.)  

Within a specific reducer, we again invoke combineReducers() to specify two reducer functions, one for 'allIds' and one for 'byId'.  

The start of klassReducer.js
```
import { combineReducers } from 'redux'

const klassReducer = combineReducers({
  byId: klassesById,
  allIds: allKlasses
})
```

So each reducer on the top level of my store is actually a combination of two reducers--byId and allIds.  This makes handling dispatch actions really simple!  One action will have to show up in both KlassesById and allKlasses.  Let's take 'ADD_KLASS' as an example.  Within KlassesById, we're dealing with the hash/object that actually contains a given klass's data.  We also need to make sure the key looks like 'klass15'.  Here's the action within the ById reducer:

```
function klassesById(state = {}, action) {
  switch(action.type) {
    case 'ADD_KLASS':
      const klassId = `klass${action.klass.id}`
      return {
        ...state,
        [klassId]: action.klass
      }
```

The same action type should trigger an action case within allKlasses, since this is an array that keeps track of all of the IDs of the klasses in my redux store ['klass15', 'klass20'...].  
```
function allKlasses(state = [], action) {

  switch(action.type) {
    case 'ADD_KLASS':
      const klassId = `klass${action.klass.id}`
      return [...state, klassId]
```

So now when I dispatch the action 'ADD_KLASS', it will add a klass object to my byID object, and add the ID to my allIds array.  This pattern takes a while to get used to, but then it makes creation so much faster.  And plus it gives me a really organized Redux store!

![](https://www.dropbox.com/s/xjakw8o6rctkmwe/my%20store%20klass%20ex.png?raw=1)


# For the next blog... 
The next challenge two big challenges to tackle were authentication on the front-end, and React Router.  The latter was especially tricky with having both teacher-only and student-only routes.  More on that next time, and thanks for reading!

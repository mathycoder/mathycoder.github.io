---
layout: post
title:      "FlexSeats with React DnD"
date:       2020-03-16 00:40:18 +0000
permalink:  flexseats_with_react_dnd
---


<img src="https://www.dropbox.com/s/cu2tv9htqhivrqe/seating-gif.gif?raw=1" alt="demo" />

One of my least favorite activites as a teacher was creating a good seating chart.  As any teacher can attest, a lot goes into choosing seats.  You have to really think through how to group your students based on their behavior, academics, and relationships.  If you teach middle or high school, you probably have multiple classes with upwards of 100+ students.  And you probably want to have flexible seats depending on different tasks.  

As my skills with React Hooks have advanced, I've been looking for an opportunity to create a new app from scratch.  Tackling the seating chart problem seemed like the perfect way to practice Hooks.  And good seating chart application can save teachers a ton of time, and potentially give them lots of ideas for best practices in grouping.

The first thing I wanted my application to have was something that made seating charts more fun to make, and that is touch screen drag and drop.  My work with the React DnD library will take up this first post on my FlexSeats app.

## Choosing a DnD Library

First, I had to select a good drag and drop package.  As far as I can tell, there are three particularly strong ones for React applications.  And I actually tried all three before finally realizing that React DnD was the way to go.  

<a href="https://react-beautiful-dnd.netlify.com/"><strong>react-beautiful-dnd</strong></a> is definitely the most 'beautiful' drag and drop library.  My Student Agendas application is built using it.  It's fantastic for sorting items within a list or multiple lists.  Alex Reardon the creator also has an awesome video series on <a href="https://egghead.io/lessons/react-course-introduction-beautiful-and-accessible-drag-and-drop-with-react-beautiful-dnd">Egghead</a> that walks you through the setup.  At first, I created each pair or group of desks as a list, and then would drag desks over to different "lists."  But this rearranged the desks in ways I couldn't control, and also didn't have the "swap" two desks capability that I was looking for.  Finally, there was no way to create a grid, which I needed for seating students in groups.

<a href="https://clauderic.github.io/react-sortable-hoc/#/basic-configuration/basic-usage?_k=1hp54o"><strong>React Sortable HOC</strong></a> is probably the easiest of all three libraries to implement, and it offers grid sorting in a way react-beautiful-dnd does not.  But the biggest issue I had with this library was the inability to move items between lists, or in my case move desks between groups.  Plus, the way the groups move objects around are preset.  So on to the last library.

<a href="https://react-dnd.github.io/react-dnd/about"><strong>React Dnd</strong></a> was ultimately the package I settled on.  This one builds on the built-in HTML5 Drag and Drop API, but adds features to make it more customizable and to allow it to work on a touch device.  While it doesn't have the neat auto-sorting graphics of the first two libraries, it has so many other options that I knew it was the best package for my application.  

## ReactDnD's `DndProvider`
Reminiscent of react-beautiful-dnd, we start our implementation of React DnD by wrapping a part of our application where we will have draggable components and droppable areas.  The <a href="https://react-dnd.github.io/react-dnd/about"><strong>docs</strong></a> go over the simplest way to set this up, which just allows for dragging with click events on desktop.  

`$ npm install react-dnd react-dnd-html5-backend --save`

After installation of the package, you can find a good place to add the following wrapper:

```
import { DndProvider } from 'react-dnd'
import Backend from 'react-dnd-html5-backend'

...

<DndProvider backend={Backend}>
...
</DndProvider>
```

However, I wanted my application to be able to handle dragging on desktops and touch dragging on mobile or tablet devices.  Unfortunately, React DnD doesn't make it easy to use both a traditional and Touch backend.

This is where I chose to use <a href="https://www.npmjs.com/package/react-dnd-multi-backend">React Dnd Multi Backend</a>, which makes the process very simple.  After installing the new package:

`npm install react-dnd-multi-backend --save`

I was able to change my new `<DndProvider>` code to this:

```
import { DndProvider } from 'react-dnd'
import Backend from 'react-dnd-html5-backend'
import HTML5toTouch from '../../backends/HTML5toTouch'

<DndProvider backend={MultiBackend} options={HTML5toTouch}>
...
</DndProvider>
```

Now we're ready to create our draggable and droppable components/elements!

## The `useDrag` Hook

One of the nicest things about the React DnD library is how it incorporates React Hooks.  It provided me a great time to think through hooks, as well as keep my new application neat by keeping all of my components functional.  I'm pretty sure this is also a disadvantage to react-beautiful-dnd, which uses class components last time I checked.

To allow my desks to drag, I imported the <a href="https://react-dnd.github.io/react-dnd/docs/api/use-drag">`useDrag`</a> hook from the 'react-dnd' library.  

```
import { useDrag, useDrop } from 'react-dnd'

...

  const [{ isDragging }, drag] = useDrag({
    item: { type: "desk", student: student },
    collect: monitor => ({
      isDragging: !!monitor.isDragging(),
    }),
  })
```

Like the `useState` hook, we use ES6 destructured assignment to name variables in the array returned by the function `useDrag`.  The first destructured variable is an object called `collectableProps`, and this object returns all of the variables we decided to 'collect' from the `useDrag` function.  More on this later, but I chose to only grab the prop `isDragging`.  The second destructured variable is `drag`, which serves as a `ref` that we use to label the element we're dragging.

Inside the `useDrag` function, we send in one object with only one required property, `item`.  The `item` is React DnD's way of letting the `drag` and `drop` elements communicate with one another. The `type` key is also required, and needs to match with the `drop` element's list of accepted types.   I also send in the `student` data.  Storing it in the item allows the `drop` element to read what student desk I'm dropping into it.

Finally, we have the `collect` property, which points to a function that has a `monitor` parameter.  `monitor` is a helpful object that has helpful functions like `isDragging()`.  Since I want my component to be aware of when an object is being dragged, I return `isDragging` in this `collect` function.  This variable gets pulled out in my `collectableProps`, so that I have access to `isDragging` anywhere in my component.  

Now we have all of the logic setup for a drag!  All we need to do is label an element in our JSX with the `drag` ref:

```
<div ref={drag} className={`desk ${hover ? 'hover' : ''} ${isDragging ? 'dragging' : ''}`}>
  <div className="desk-items">
    <div className="first-name">{student.firstName}</div>
    <div className="last-name">{student.lastName}</div>
  </div>
</div>
```

This is my draggable desk.  We set `ref={drag}`, making this our draggable element.  Then I also take advantage of my collectable prop `isDragging`, using it to create a `dragging` class whenever this desk is being dragged.  Whenever a desk is dragged in my application, I set that dragged desk to a light gray, as you can see in the opening gif. 

## The `useDrop` Hook 
So now that I can drag my desks around, I need to be able to drop them into certain areas.  Granted, setting up where my drop `<div>`s would live so that they fit neatly into pairs or groups and adjusted for different screen sizes... that's a whole other topic for CSS grid.  But in terms of the drop functionality, this is where React DnD's second major hook <a href="https://react-dnd.github.io/react-dnd/docs/api/use-drop">`useDrop`</a> comes into play:

```
const [{ hover }, drop] = useDrop({
    accept: "desk",
    collect: monitor => {
      return ({ student: student, hover: monitor.isOver() })
    },
    drop: (item, monitor) => {
      swap(klass, student, item.student, type)
    },
  })
```

Okay, we have our same hook destructured assignment.  This time, I collected `hover`.  If we look at the collect function, we see that we set `hover: monitor.isOver()`, which allows me to highlight desks in yellow whenever a dragged element is hovering over a given desk.  I also have `drop`, which is the `ref` we'll use to specify our droppable element.  

Also notice that the type `desk` is being accepted, which is how we labeled the type in `useDrag`.  

And finally, when we drop a dragged object into this drop area, the callback function `drop` is called.  This function has two parameters, `item` and `monitor`.  `item` is what we created in the `useDrag` hook, and monitor gives us helpful methods.  I used my own function `swap` to initiate the front- and back-end logic to swap the dragged student desk with the one in the dropped area.

## Dragging and Dropping Desks
 Let's look at all of the JSX rendered by the `<Desk/>` component.  Believe it or not, this is all we need to attach drag and drop functionality to each desk.

```
  return (
    <>
      <div className="desk-drop-area" ref={drop}>
        <div ref={drag} className={`desk ${hover ? 'hover' : ''} ${isDragging ? 'dragging' : ''}`}>
          <div className='groove' />
          <div className="desk-items">
            <div className="first-name">{student.firstName}</div>
            <div className="last-name">{student.lastName}</div>
          </div>
        </div>
      </div>
      {index % 2 === 1 ? <div className="gap"></div> : null}
    </>
  )
}
```

First, I wanted to make my droppable areas the exact same size as my desks, since I'd always be dropping a desk on top of another desk, blank or otherwise.  So we have our initial `<div>` which has the `ref={drop}`, and then we have an inner `<div>`, which has the `ref={drag}`.  Picture a droppable square that fits a draggable desk exactly inside of it.  Voila!  It is clean!

Where things get meta is by thinking through: who is the `student` listed here?  Is it the dragged or the dropped?  Well, the thing is, each seating chart has 32 of these `<Desk />` components.  So my short answer is: both!  Each `<Desk />` has the ability to be dragged, and has a droppable area for another desk to be dropped into.  That's kinda cool!

<img src="https://media.giphy.com/media/96FloqY8aYqwo/giphy.gif" style="display:block; margin: 0 auto;"/>


## Conclusion
Well, that's a brief introduction to drag and drop packages, why I chose React DnD, how to set up functionality on desktop and mobile, and it's main two hooks `useDrag` and `useDrop`.  Thinking through what we need to collect and what information needs to be communicated between dragging and dropping was tricky, but I'm happy with how it came together.  

The last major hiccup was getting the desk I dragged to actually show up on mobile.  This involved an extra set of steps involving creating a `<Preview />`, which is pretty well explained on <a href="https://www.npmjs.com/package/react-dnd-multi-backend">react-dnd-multi-backend</a>.  

So much more has gone into creating FlexSeats, and I don't foresee leaving this application build anytime soon.  Maybe I'll create a version in React Native so that FlexSeats can be an app in the App Store?  We'll see, but for now, it's been a blast developing a better way to help teachers group their students with the help of my new skills as a coder.  

As always, happy coding!


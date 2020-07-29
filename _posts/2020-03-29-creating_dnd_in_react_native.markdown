---
layout: post
title:      "Creating DnD in React Native"
date:       2020-03-29 16:16:15 -0400
permalink:  creating_dnd_in_react_native
---

<img src="https://www.dropbox.com/s/045jkso4awu5b3x/iphone%20demo.gif?raw=1" alt="iphone demo" style="display: block; margin: 0 auto;" />

## Intro

My conversion of FlexSeats from ReactJS to React Native continued this week.  After converting my CSS Grid desk arrangement to FlexBox, I was pretty happy with how a seating chart adapts to different screen sizes.  

But when it was time to start adding drag and drop functionality to my React Native application, I again hit a roadblock.  ReactJS may have a ton of DnD packages on [npmjs](https://www.npmjs.com/) and [react.parts](https://react.parts/), but React Native has next to none.  I needed my drag and drop functionality to be able to drag a desk on top of another desk, drop it into a drop zone, and initiate a swap.  Pretty much every existing package does some form of list sorting, which isn't what I need.  

What I really needed was something to mimic the style of react-dnd, which is what I used on my desktop version of [FlexSeats](https://flexseats.herokuapp.com/).  I needed to touch a desk, drag a top-level copy of it on top of any other desk, and initiate a seat move/swap.  

There are a few helpful resources of other developers navigating somewhat similar projects.  The top hit on google is [Creating a Drag and Drop Component With React Native](https://blog.reactnativecoach.com/creating-draggable-component-with-react-native-132d30c27cb0/).  So I used this as a starting point, and then relied heavily on documentation for [PanResponder](https://reactnative.dev/docs/panresponder) and [Animated API](https://reactnative.dev/docs/0.51/animations), both of which are built into React Native.  

## PanResponder and Animated API
To get drag and drop to happen on a phone, we need both the recognition of touch gestures and the ability to animate components.  That's where both PanResponder and Animated API come into play, and from the documentation, both tend to be set up together.  

I decided to set up my panResponder at the parent level above each desk.  Both of these tools (and React Native) are new to me, so I'll do my best to explain it given my current level of understanding.  Also, I created everything using functional components and hooks:

```
  const pan = useRef(new Animated.ValueXY()).current;
	...
  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderGrant: (e, gesture) => {
        setDraggedStudent(e._targetInst.memoizedProps.student)
        pan.setOffset({
          x: pan.x._value + gesture.x0 - cloneLocationRef.current.x,
          y: pan.y._value + gesture.y0  - cloneLocationRef.current.y
        });

      },
      onPanResponderMove: Animated.event(
        [
          null,
          { dx: pan.x, dy: pan.y }
        ]
      ),
      onPanResponderRelease: (e, gesture) => {
        setDraggedStudent(null)
        pan.setOffset({ x: 0, y: 0 })
        pan.setValue({ x: 0, y: 0 })
      }
    })
  ).current;
```
	
My starter code for this came from [React Native: PanResponder](https://reactnative.dev/docs/panresponder).  `  const pan = useRef(new Animated.ValueXY()).current;` creates a reference to a new Animated value.  This tracks an (x,y) position that will be linked to my desk that is dragging, which will be linked to an `<Animated.View>` component.  As the desk drags, this value will change dynamically, triggering a translation animation to make the desk move across the screen.  

The `panResponder` responds to the touch gesture of a user, and will update the pan translation dynamically.`onMoveShouldSetPanResponder` is an initializer of sorts; when an animated view is touched and a drag about to begin, panResponder will start a process of tracking the touch gesture.  

`onPanResponderGrant` takes a function as a value, which is where we program the initial rules of the drag.  

```
onPanResponderGrant: (e, gesture) => {
        setDraggedStudent(e._targetInst.memoizedProps.student)
        pan.setOffset({
          x: pan.x._value + gesture.x0 - cloneLocationRef.current.x,
          y: pan.y._value + gesture.y0  - cloneLocationRef.current.y
        });
```

The main thing this function normally does is set the initial pan offset, and normally, it's zero.  Think about it: when we touch a draggable object on the screen, we don't want to initially translate it across the screen at all; we just want it to move with the finger.  

However, I needed this level of customization because I DON'T want the touched desk to actually be dragged at all; I want to drag a clone.

<img src="https://media.giphy.com/media/kHZ5zz8Q1SSf7TM7Mp/giphy.gif" alt="paul rudd clone" style="display: block; margin: 0 auto;" width="350px" /> 

Why use a clone?  I'll get to that in a minute.  But first, how can we make the cloned desk be dragged instead of the one the user touches?  First, I need to only animate the cloned desk, which I'll also show later.  But more importantly, I need to move the cloned desk dynamically to be right on top of the actual desk the user touches, right as they touch it.  Let's look at `pan.setOffset()` more closely, and a horribly drawn diagram I made to help clear this up:

`x: pan.x._value + gesture.x0 - cloneLocationRef.current.x,`

<img src="https://www.dropbox.com/s/w9sgpvxcl27qv83/clone-diagram2.png?raw=1" alt="drag diagram" style="display: block; margin: 0 auto;"  width="520px" />

We have the initial value of `pan.x`, which is essentially delta x, or how x changes during the drag.  Since this is the beginning of the drag, it is currently 0.  To move the cloned desk under the finger of the user, I need to do two things.  First, I need to subtract the x-position of the cloned desk, which is `cloneLocationRef.current.x`, to get it to x : 0.  Then I need to add the current x-position of the finger, which is `gesture.x0`.  This means at the start of a drag, the clone desk becomes the desk being dragged, showing up under the user's finger dynamically.  Pretty cool, right? 

My panResponder has two other methods:
```
 onPanResponderMove: Animated.event(
        [
          null,
          { dx: pan.x, dy: pan.y }
        ]
      ),
      onPanResponderRelease: (e, gesture) => {
        setDraggedStudent(null)
        pan.setOffset({ x: 0, y: 0 })
        pan.setValue({ x: 0, y: 0 })
      }
```

`onPanResponderMove` uses an Animated event to continually update that delta x and delta y value, changing it dynamically with the user's finger drag.

`onPanResponderRelease` is where we can customize what happens at the end of a drag.  Aside from my own state management that tracks which student is being dragged, I needed to set both the pan offset and pan values to zero.  This took me a LONG time to figure out.  Of course, it makes sense to make the value of the pan equal to zero.  This starts from zero at the beginning of each drag.  But all of that offsetting I did to make the clone jump to the dragged desk's position ... that offset also needs to be reset at the end of a drag.  These two lines will jump the clone back to its original position off the screen.

### Why bother cloning at all?
I honestly think a lot of humans ask this question all the time.  But in this case, cloning gave me control of a few problems I was having.  The main problem involved z-indexing in React Native.  My dragged desk would be in front of the desks rendered before it, and behind all other desks.  This problem is lightly documented online; here are some resources I used to try and solve the problem.
* https://stackoverflow.com/questions/40326822/drag-drop-and-swap-items-animation
* https://github.com/fangwei716/30-days-of-react-native/blob/development/view/day18.js
* https://github.com/facebook/react-native/issues/698

The most common solution suggested was to make all of the draggable objects siblings, set their positions to 'absolute', and then adjust their zIndex values dynamically.  But with so much of my layout depending NOT on absolute values, and with desks being nested in both row and pair `<View>`s, this didn't seem like an option.  That's why I chose to CLONE!

## More on that cloned desk
Inside my parent component, I decided to render my `<CloneDesk />` after each of my actual desks.  This allows it to be dragged OVER everything else:

```
<View style={styles.PairSeatingChart}>
        {renderDeskRows()}
      </View>
      <CloneDesk
        pan={pan}
        panResponder={panResponder}
        student={draggedStudent}
        setCloneLocation={setCloneLocation}
      />
	```
	
I needed to send in `pan` (the AnimatedXY ref), `panResponder`, `draggedStudent` (which is where I keep track of which student is being dragged in the local state, and `setCloneLocation`.  This last setter state method is what allows us to subtract the clone's location to move it to x = 0 earlier.  
	
Let's jump inside `<CloneDesk />`.

```
import React, { useState, useRef } from 'react'
import { Text, View, StyleSheet, Animated, measure } from 'react-native'

const CloneDesk = ({ student, index, pan, panResponder, setCloneLocation  }) => {
  const refContainer = useRef(null)

  const panStyle = {
          transform: [{ translateX: pan.x }, { translateY: pan.y }]
        }

  const measureMe = (nativeEvent) => {
    if (refContainer){
      refContainer.current.getNode().measure((fx, fy, width, height, px, py) => {
        setCloneLocation({x: px + width/2, y: py + height/2})
      })
    }
  }

  return (
    <View style={{position: 'absolute', left: -50, top: -50}}>
      <Animated.View
        style={[student ? styles.deskStyle : styles.hiddenStyle, panStyle]}
        {...panResponder.panHandlers}
        ref={refContainer}
        onLayout={({ nativeEvent }) => measureMe()}
      >
          <View style={styles.grooveStyle}></View>
          <View style={styles.deskItemsStyle}>
            <Text style={styles.deskItemsText}>{student ? student.firstName : ''}</Text>
            <View style={styles.ratingsStyle}>
            </View>
          </View>
      </Animated.View>
    </View>
  )
}

export default CloneDesk
```

Definitely lots to look at here.  First, let's look at the return JSX statement.  I needed to make sure the outer `<View>` didn't affect the layout of the rest of the application, so I gave it an absolute position off the screen.  Within that is our `<Animated.View>`, which allows for dynamic translation across the screen.  To make it responsive to changes to the panResponder, I spread the `{...panResponder.panHandlers}` into the component.  

I also needed to tell the animated view how much to translate.  So I set up `panStyles`:

```
  const panStyle = {
          transform: [{ translateX: pan.x }, { translateY: pan.y }]
        }
```

This will take the current value of `pan` and apply a translation transformation to the x and y values of my view.  

### Sending up the clone's location

The other big to-do here is to send the location of the cloned desk up to the parent, so that the offset works correctly.  Grabbing the location of a rendered node in React Native is different than on the web.  To do it, I scoured the internet to find the `.measure()` function, which you can call on any DOM node, as well as the callback `onLayout`.  The latter is called after a DOM node is placed on screen, which by then should have it's location. I also created the ref `refContainer` and placed it on the `<Animated.View>`.  

Okay, so when the screen loads, `onLayout` is trigged, which runs my helper function `.measureMe()`.  

```
const measureMe = (nativeEvent) => {
    if (refContainer){
      refContainer.current.getNode().measure((fx, fy, width, height, px, py) => {
        setCloneLocation({x: px + width/2, y: py + height/2})
      })
    }
  }
```

This function grabs the current node being pointed at by our reference, and runs that helpful method `measure()` on it.  The last two parameters are `px` and `py`, which are the location on the screen of the top left corner of the clone desk.  Since I want the center, I also add half the width and height to it.  So now the parent knows the location of the cloned desk!

## One final piece of the puzzle
This last piece may be a little hacky, but hey, it's working for now!  I needed some way to alert the panResponder that one of the desks was trying to be dragged, WITHOUT actually letting anything but the clone be dragged.  So inside each of my 32 `<Desk />` components, I have this:

```
<View>
      <View
        style={[styles.deskWrapperStyle, floatingStyle]}
        {...panResponder.panHandlers}
        student={student}
      >
          <View style={styles.grooveStyle}></View>
          <View style={styles.deskItemsStyle}>
            <Text style={styles.deskItemsText}>{student.firstName}</Text>
          </View>
      </View>
    </View>
```

My hack is this: I've attached the panHandlers to a `<View>` that ISN'T animated.  This means that touching one of the 32 desks will trigger the panResponder, but will only let the clone be dragged across the screen.  When I switch this to an `<Animated.View>`, all 32 desks and the clone start moving across the screen.  

Note: I also use memoized props here.  Notice inside my `<View>` I have `student={student}`.  This is the line that lets me set the dragged student at the parent level.  I can't simply set the state here because then I'd be setting it for all 32 desks.  Instead, the panResponder gets triggered, and grabs the student from that particular node out of the props.  

## Conclusion
It was honestly a journey to get to something that looks so simple: dragging a desk across the screen.  And I'm not even done, since I need to create a droppable area to allow seats to change!

But after months of relying on React Drag and Drop packages, it does feel pretty cool to create the functionality from scratch.  

As always, happy coding!

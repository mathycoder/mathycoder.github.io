---
layout: post
title:      "Creating DnD in React Native, part II"
date:       2020-04-05 19:32:45 +0000
permalink:  creating_dnd_in_react_native_part_ii
---

<img src="https://www.dropbox.com/s/dy0ze7qyk8tohyt/flexseats-demo3.gif?raw=1" alt="flex-seats demo" width="600px" style="display: block; margin: 0 auto" />

Whew, I did it!  

Creating a version of drag and drop for React Native was a bigger project than I initially intended.  At one point, I almost backed out of DnD completely in favor of tapping the desks to make them swap.  But I'm really happy I hung in there and finished adding DnD to my app.

I took on this project after realizing React Native doesn't have very many DnD packages, and the ones I love in ReactJS aren't compatible with Native.  In [part I](https://mathycoder.github.io/creating_dnd_in_react_native), I explored using React Native's Pan Responder and Animated APIs to drag a desk across the screen.  I ended up creating a clone desk that replaces the desk being dragged.  My idea came from ReactJS's react-dnd, which uses a drag layer on touch devices when dragging.  

In this part, I'll explore the creation of drop zones to allow desks to be dropped and seats to be moved!

## Defining the Drop Zones, failed attempt

I knew that I needed some way to determine whether or not the dragged desk was over a drop zone.  I decided to use the absolute position of each of my 32 desks.  For each desk, I would keep track of its topLeft, topRight, bottomLeft, and bottomRight locations, and store it in my Redux store.  

To make this happen, I first tried the strategy of grabbing each desk's current location using React Native DOM node methods.  To do so, inside all 32 of my `<Desk />`s  I created a reference to the given element. 

```
ref={refContainer}
onLayout={({ nativeEvent }) => myMeasure()}
```

When any element is first rendered on the screen, `onLayout` is called, which invokes my callback function `myMeasure()`. 

```
  const myMeasure = (nativeEvent) => {
    if (refContainer){
      window.setTimeout(() => {
        refContainer.current.getNode().measure((fx, fy, width, height, px, py) => {
          setCloneLocation({x: px + width/2, y: py + height/2})
        })
      }, 0)
    }
  }
```

We can use the current reference to grab the current x and y screen coordinates `px` and `py` of the desk, and then manipulate it a bit in the store to get the coordinates of all four corners.  

When I got this working, I rendered the borders on the DOM to make sure they closely bordered each desk, thus defining the drop zones.  At this point, I noticed something strange happening.  Each time I reloaded my screen, the borders moved.  They never stuck closely around each desk, and sometimes they weren't even close.  

After lots of troubleshooting, I decided to abandon this strategy for defining my drop zones.  I needed something more consistent, and this strategy seemed like a dead end.  As to why, my guess is using `onLayout` can be glitchy, and that sending 32 dispatches at once is also glitchy. So I devised a second strategy.

## Defining the Drop Zones, successful attempt!
<img src="https://www.dropbox.com/s/iphdzirusf667sw/desk%20layout.png?raw=1" alt="desk diagram" width="400px" style="display: block; margin: 0 auto" />

When initially laying out my desks, I wanted them to look good on different sized mobile screens.  This led me to move away from absolute positioning; what absolute position looked nice on an iPhone 5 might look horribly cluttered on an iPhone X.   Instead I took advantage of the FlexBox styling implicit to every layout in React Native.  The key attribute is `justifyContent: 'space-around'`.  While the sizes of my desks look the same on each mobile device, the space around each desk changes based on the screen size.  So theoretically, if we grab the dimensions of the screen using React's `Dimension`, we can use some math to determine the location of each desk on the screen!

So I created a Redux action creator to send the dimensions of the given window to a reducer function.

`setSeatLocations(Dimensions.get('window').width, Dimensions.get('window').height)`.

And then I used this reducer logic, which I'd love to break down as a former math teacher ;)  The entirety of the function is at the end of this section.

```
function setSeatLocations(stateCopy, screenWidth, screenHeight){
  const deskWidth = 65
  const deskHeight = 52
  const marginViewVertical = 20
  const paddingHorizontal = 20
  const paddingVertical = 20
	...
```
First, I created the constants.  I padded the view horizontally and vertically, and used constant desk widths and heights  I also have a header at the top of the screen, which is the additional marginVertical.

```
const marginHorizontal = (screenWidth - deskWidth*8 - paddingHorizontal) / 8
const marginVertical = (screenHeight - deskHeight*4 - paddingVertical - marginViewVertical) / 5
```

Determining `marginHorizontal` required some math.  If you look at the diagram I drew above, there are horizontal spaces between the desks labeled in green.  I figured out that there are 8 of these going horizontally, which are determined using `justifyContent: space-around`.  To figure out their exact lengths, I subtracted away everything constant from the screen width.  So we start with the screen width, and then have to subtract away the widths of 8 desks going across.  We also need to remove the horizontal padding of the seating chart on the screen.  What we're left with is all of the extra horizontal space.  Divide this by the 8 green margins, and we know how long each horizontal margin should be!  I used similar reasoning for the vertical distances in red.  

Now that I knew the lengths of these margins, I started to determine the locations of each desk on the screen.  

```
for (let key in stateCopy){
    const seat = parseInt(key.split("seat")[1])
    const row = Math.floor(seat / 8)
    const col = seat % 8
	```
	
	In my state, I had `desk0`, `desk1`, etc. all the way up to `desk31` already preset.  So I looped through all of the keys in the state, grabbing that seat number in `const seat`.  The desks are numbered horizontally, so 0-7, 8-15, etc.  To find each row, I just needed to divide the seat number by 8 and chop off the remainder.  To find each column, I just needed the remainder, so I used `%`.  
	
Still inside the for-loop, now I was ready to find the x and y offsets for each desk.  Let's just look at the x-offset, generalized for any desk:

```
...
    const relMarHor = marginHorizontal
                                + Math.floor(col / 2) * marginHorizontal*2
                                + col * deskWidth
```

First, I figured that each desk would have at least one marginHorizontal (green margin).  To determine how many more green margins to add, for every two desks I added another margin 2 margins (again, the diagram helps to see why).  Finally, I took the column number and multiplied it by the desk width; that's because each desk before it would offset the horizontal position by their width.  So a desk in column 7 would have 7 desks before it, so `7 * deskWidth`.

With `relMarHor` and `relMarVer` giving us the relative offset for any particular desk, I was ready to set the state for each desk location inside the for-loop .I used the desk width and height to trace out the rectangle defining the bounds of the desk.

```
    stateCopy[key].topLeft = { x: relMarHor, y: relMarTop }
    stateCopy[key].topRight = { x: relMarHor + deskWidth, y: relMarTop }
    stateCopy[key].bottomLeft = { x: relMarHor, y: relMarTop + deskHeight }
    stateCopy[key].bottomRight = { x: relMarHor + deskWidth, y: relMarTop + deskHeight }
    stateCopy[key].center = { x: relMarHor + deskWidth/2, y: relMarTop + deskHeight/2 }
```

While it may seem a little math heavy, this second strategy was better than grabbing the DOM locations for my 32 desks.  For one, it only needed to send one dispatch to the Redux store to update all 32 seat locations.  For another, the only external inputs it needed were the length and width of the screen.  This makes it more reliable in producing accurate drop zones.

Here is the entire function. 
```
function setSeatLocations(stateCopy, screenWidth, screenHeight){
  const deskWidth = 65
  const deskHeight = 52
  const marginViewVertical = 20
  const paddingHorizontal = 20
  const paddingVertical = 20

  const marginHorizontal = (screenWidth - deskWidth*8 - paddingHorizontal) / 8  // 47
  const marginVertical = (screenHeight - deskHeight*4 - paddingVertical - marginViewVertical) / 5

  for (let key in stateCopy){
    const seat = parseInt(key.split("seat")[1])
    const row = Math.floor(seat / 8)
    const col = seat % 8

    const relMarHor = 10 + marginHorizontal
                                + Math.floor(col / 2) * marginHorizontal*2
                                + col * deskWidth
    const relMarTop = 13 +marginVertical
                                + row * marginVertical
                                + row * deskHeight

    stateCopy[key].topLeft = { x: relMarHor, y: relMarTop }
    stateCopy[key].topRight = { x: relMarHor + deskWidth, y: relMarTop }
    stateCopy[key].bottomLeft = { x: relMarHor, y: relMarTop + deskHeight }
    stateCopy[key].bottomRight = { x: relMarHor + deskWidth, y: relMarTop + deskHeight }
    stateCopy[key].center = { x: relMarHor + deskWidth/2, y: relMarTop + deskHeight/2 }
  }
  return stateCopy
}
```

## Sensing a Drag Over
During a drag, I wanted a drop zone to light up in yellow whenever the dragged desk hovers over the zone.  Since I had each of the drop zones in my Redux store, I figured I would loop through the list during each incremental translation of the desk.  Changing things while an object is being dragged happens in the Pan Responder's `onPanResponderMove()` method.  

```
onPanResponderMove: (e, gesture) => {
        const over = desksRef.current.allIds.find(seatId => {
          const seat = desksRef.current.byId[seatId]
          if (seat.topLeft && seat.topRight && seat.bottomLeft && seat.bottomRight){
            return (gesture.moveX > seat.topLeft.x && gesture.moveX < seat.topRight.x &&
                    gesture.moveY - 35> seat.topLeft.y && gesture.moveY - 35 < seat.bottomLeft.y)
          } else {
            return false
          }
        })
        setOverDesk(over)
...
```

Basically, I use the `gestureState` parameter to grab the current x- and y-coordinates of the finger on the screen.  Then I checked to see if it was in the bounds of any of my desk drop zones.  I tracked this in the local component state variable `overDesk`.  

Notice I don't simply iterate through `desks`, but instead iterate through `desksRef.current`.  I ended up learning a lot about the `useRef()` here.  When the pan responder is created, `desks` is captured in a closure.  But the values in `desk` change when the reducer function in the previous section gets called asynchronously.  Long story short, when the `panResponder` is first created, we don't have the drop zone locations.  To give the `panResponder` access to the asynchronously determined locations, I took advantage of the `useRef()` hook, which allowed me to access a current reference to the updated `desks` from within a closure.  To set this up, I put this at the beginning of my component: 

```
  const desksRef = useRef(desks)
  
  useEffect(() => {
    desksRef.current = desks
  }, [desks])
```

The `useEffect()` hook runs a function whenever theres a change to `desks`, which is exactly what I wanted.  So the `desksRef` contains a reference to the current `desks`, which is where I stored the drop zones.

So, as the dragged desk hits the drop zones, `setOverStudent` updates my state variable `overStudent`, which I send in to each `<Desk />`, highlighting the contained student if they match `overStudent`.  

And now for the final step:

## Initiating a Swap or Move
```
onPanResponderRelease: (e, gesture) => {
        const seatNumber = overDeskRef.current ? parseInt(overDeskRef.current.split("seat")[1]) : null
        const overStudentId = studentsRef.current.allIds.find(stId => {
          const student = studentsRef.current.byId[stId]
          return student.seatPair === parseInt(seatNumber)
        })

        const currentStudent = e._targetInst.memoizedProps.student
        const overStudent = overStudentId ? studentsRef.current.byId[overStudentId] : null

        if (overStudent) {
          swap(klass, currentStudent, overStudent, 'pair')
        } else if (overDeskRef.current) {
          newSeat(klass, currentStudent, seatNumber, 'pair')
        }

        setDraggedStudent(null)
        pan.setOffset({ x: 0, y: 0 })
        pan.setValue({ x: 0, y: 0 })
        setOverDesk(null)
```

`onPanResponderRelease` is the `panResponder` method that initiates every time all touch gestures have ended.  I take advantage of this fact to determine if we're dropping the desk on a pre-existing desk.

`const overStudentId` loops through the students from the Redux store to see if our `overDesk` number matches a given student's seat.  I set this equal to `overStudent`.  I also grab the `currentStudent` using the memoized props from part I.

Finally, I initiate either `swap` or `newSeat`, both action creators that initiate a back- and front-end state update for the new seats!  

## Conclusion
This task was no joke!  But it marks the first time I've created Drag and Drop functionality from scratch.  It definitely gives me an appreciation for JavaScript package creators, and also gives me a good idea of what creating custom functionality would look like.

Now with my DnD functionality, I look forward to continue building out this React Native app, and eventually getting an application on the App Store!

As always, happy coding!  



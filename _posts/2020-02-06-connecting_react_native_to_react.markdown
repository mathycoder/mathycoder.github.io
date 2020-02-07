---
layout: post
title:      "Connecting React Native to React"
date:       2020-02-07 00:03:43 +0000
permalink:  connecting_react_native_to_react
---


As an educator, one strategy to help students solidify their knowledge of a new topic is to help them make connections. The more ways we can make connections to a new idea, the deeper an understanding we can develop.  Here's a snippet from Van De Walle's *Elementary and Middle School Mathematics* that illustrates the concept:

![](https://www.dropbox.com/s/4zv14tfmqlm1ert/van%20de%20walle.png?raw=1)

Having received a solid education in ReactJS through the Flatiron School, I decided to try learning React Native.  This framework allows developers to use a single framework to create applications for iOS and Android.  To explore this new framework, I picked a Udemy course.  The course is great -- [Udemy - The Complete React Native + Hooks Course](https://www.udemy.com/course/the-complete-react-native-and-redux-course/) -- but like many online coding courses, it leaves the connection making to the learner.  So let's make some connections!  Note: many of the following code snippets are from the course.

## `<Text> && <View>`

The first thing I learned were some of my friends from the HTML world -- `<div>`, `<p>`, `<h1>` -- were gone in React Native.  After mourning the loss of said friends (honestly, they've been with me from the start of my coding journey), I began to dive into their replacements.  React Native comes with some *primitive elements*.

![](https://www.dropbox.com/s/zy5vlmxvuv23h5z/comparison1.png?raw=1)

HTML5 elements have a lot of preset stylings.  The paragraph and the header tags all come equipped with initial font sizes, font weights, and preset padding.  React Native primitive elements have next to no preset stylings, and also seem to condense many HTML5 elements together.  `<Text>` is the catch-all for all header and `<p>` tags, and it leaves the styling to you.

Similarly, the `<View>` element is a catch-all for organizational tags like `<div>` and `<span>`.  And again, the inline vs new-line stylings are gone, leaving it up to the coder to set in styling.  Speaking of styling...

## Goodbye CSS 

While losing my `<div>`s and `<p>`s felt like losing old friends, losing CSS styling felt like losing a first born.  But worry not; the React Native `StyleSheet` is a pretty good replacement.  

```
const styles = StyleSheet.create({
  containerStyle: {
    marginLeft: 15
  },
  imageStyle: {
    width: 250,
    height: 120,
    borderRadius: 4,
    resizeMode: 'cover',
    marginBottom: 5
  },
  nameStyle: {
    fontWeight: 'bold'
  }
})
```

There don't seem to be classes used in React Native.  Instead, you create a styles object using StyleSheet.create().  Each key/property in this object is where you apply more traditional CSS properties.  Once you've created the styles you like, you can apply them to an element in your JSX like this:

`<Text style={styles.nameStyle}>{result.name}</Text>`

There's a lot of semantic variation between CSS properties and the ones that work similarly in a `StyleSheet`.  Some are just semantic; for example, `object-fit` is the CSS property that lets an image be stretched, contained, or to fill a given height and width; in a `StyleSheet` the property is `resizeMode`.  Other changes are more helpful.  A typical pattern I use when applying margins is to make both the left and right margins the same.  The property `marginHorizontal` takes care of this, as well as its sister `marginVertical`.  

For fans of Flexbox over CSS Grid, you're in luck: *everything in React Native uses Flexbox!*  Every `<View/>` container is treated like a parent-level flex container, with properties like `justifyContent` and `alignItems` working at the parent level, as well as `flex` or `alignSelf` working at the child level.  Considering the varying sizes of mobile devices, using Flexbox over CSS Grid makes a lot of sense to me.  

## `<FlatList />`

Sometimes it feels like React Native is capitalizing on patterns that ReactJS uses routinely.  One functional programming pattern React encourages is mapping over an array of state values.  For example, as a user adds or removes blog posts, an index page would use the higher-order .map() function to return JSX to render that list.  I used this pattern so often in my previous React/Redux project that it began to feel like a natural part of React.

And now it is!  `<FlatList />`, another React Native primitive element, takes every detail of mapping over state to render JSX and embeds it into props you need to pass into it to make it work:

```
<FlatList
    horizontal={true}
    showsHorizontalScrollIndicator={false}
    data={results}
    keyExtractor={result => result.id}
    renderItem={({ item }) => {
        return (
            // JSX to render
     )}}
/>
```

`<FlatList>` requires three things.  
1. A data prop.  Instead of doing `results.map(...)`, we instead set the data prop equal to our array.
2. A renderItem prop.  This is essentially our callback function that would be the argument passed into .map().  There's some nested brackets in that function, which take advantage of destructured assignment to get the actual item we want to map into JSX.
3. A key for every JSX element.  While we could do this manually, it makes more sense to use the keyExtractor, which will create a key based on some property of the current item.  

It's kind of nice that certain props are required.  In this way, React Native will catch your errors early, and even tell you which prop you're missing.  Also, there are some helpful props that can flip the list to be vertical or horizontal, as well as the ability to deactivate scrollbars.  

## Routing 

React Router for ReactJS felt very different than any routing I was used to with Ruby on Rails.  Rails is "Routes First", with everything stemming naturally from that initial route redirecting to a controller.  This was in contrast to React Router, which allowed you to nest routes within components.  For my last program, some routes were deep within my component stack.

React Native uses `react-navigation-stack`, which seems much more aligned to Rails.  Below is our top level `App.js` component, which sets the stage for the entire application.

```
import { createAppContainer } from 'react-navigation';
import { createStackNavigator } from 'react-navigation-stack';
import SearchScreen from './src/screens/SearchScreen'
import ResultsShowScreen from './src/screens/ResultsShowScreen'

const navigator = createStackNavigator({
  Search: SearchScreen,
  ResultsShow: ResultsShowScreen
}, {
  initialRouteName: 'Search',
  defaultNavigationOptions: {
    title: "Business Search"
  }
})

export default createAppContainer(navigator)
```

`react-navigation-stack` comes with a function called `createStackNavigator()`.  This function takes two objects as arguments.  While the second is just for router settings, the first is where we set up our routes.  For example, `Search: SearchScreen` is creating a route to the component `<SearchScreen>`, and giving that route a name called `Search`.  This is really similar to Rails' route helpers (e.g. `search_path`).  

Interestingly, React Native (like ReactJS) needs to render something when it loads an app.  There's no render() or return ( ) in this top-level component.  Instead, we use `createAppContainer()`, a function that will return a version of our navigator that React Native will interpret as a component, as it decides which component to render.  

Navigating to another screen within the application involves a few more steps.  Basically, every top level route has a `navigation` property in its props.  To navigate to the `<ResultsShowScreen` for example, you might do `props.navigation.navigate('ResultsShow', { id: 5 } )`.  Again, this is very similar to Rails -- we use our version of a Rails URL helper, and send in an argument when that route needs additional data.  

## Is it all different?

If you're reading this and discovering React Native for the first time, it might seem so.  But so much of React Native looks and behaves exactly like React.  One area that seems to be nearly identical is state management.  Take a controlled form component.  

```
const SearchBar = ({ term, onTermChange, onTermSubmit }) => {
  return (
    <View style={styles.backgroundStyle}>
      <TextInput
        autoCapitalize="none"
        autoCorrect={false}
        style={styles.inputStyle}
        placeholder="Search"
        value={term}
        onChangeText={onTermChange}
        onEndEditing={onTermSubmit}
      />
    </View>
  )
}
```

While `<TextInput>` is a new React Native primitive element, the concepts behind it are identical to ReactJS.  This input field has a placeholder.  It has an initial value, `term`, which is controlled in a parent component's local state.  It has an event listener called `onChangeText` (vs. `onChange` in ReactJS), where a callback function sets our new state value for `term`.  And instead of `onSubmit` for the form, we have `onEndEditing`, an event listener that lets you do what you want with the final form data once the user clicks enter or return.  

The similarities continue.  In both ReactJS and React Native, you can manage state in functional components with React Hooks (which this course goes over beautifully), which allows you to use local state values (`useState`), localized reducers (`useReducer`), or incorporate Redux for state management.  Similarly, you can use class components, which gives you lifecycle methods, the `setState()` function, and again the ability to connect to Redux.  

## From here

Thanks for coming along my connections journey.  I'm currently at a crossroads: create my own React Native app to solidify my learning, or continue down the path of guided instruction.  If you're new to React Native, check out the [Udemy course](https://www.udemy.com/course/the-complete-react-native-and-redux-course/), and good look on your learning path!




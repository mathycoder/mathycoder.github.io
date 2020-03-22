---
layout: post
title:      "FlexSeats Part II - React Native Front-End"
date:       2020-03-22 13:42:45 +0000
permalink:  flexseats_part_ii_-_react_native_front-end
---

As I've grown more and more excited working on FlexSeats -- my application to help teachers create flexible seating charts by grouping students heterogenously or homogenously via behavior and academic scores -- I've started to focus on exploring mobile technologies.  My web app https://flexseats.herokuapp.com works very well on a mobile device, after hours of tinkering with media queries, FlexBox, and CSS Grid.  But the urge to create a FlexSeats app on the App Store has been great lately, and not just to make me famous ;)  Functionally, I'd like my app to lock in landscape mode when teachers are moving seats, and I'd like the URL menu bar to not exist.  I'd also love for teachers to always be logged in, so they can just tap their app and start working with their seats.  

After exploring [this React Native Udemy course](https://www.udemy.com/course/the-complete-react-native-and-redux-course/), I had "made" a few React Native apps... that is, I had followed the instructor's steps exactly and followed his reasoning.  So this seemed like an opportunity to really start learning and try to recreate my FlexSeats React front-end with React Native, and begin my quest towards an app on the App Store and world domination.  

<img src="https://media.giphy.com/media/OLHy9ERaFUvzW/giphy.gif" style="display: block; margin: 0 auto;" />

## The power of separating layers your my stack
I created FlexSeats the web application using a React/Redux front-end and a Rails API backend.  So as I've started work on the React Native front-end, I've realized how powerful separating layers in your stack can be.  At first I thought I needed to create a totally new app from scratch, so that I'd have two apps: FlexSeats for mobile, and FlexSeats for the web.  

But why not just tap into my Rails API backend?  Both my web front-end and my mobile front-end with interact with the backend in the same way: sending HTML requests to the API backend to fetch and modify data.  Plus, as I work on the front-ends, any changes to the backend can happen in the exact same place (read: DRY).  

That's only the beginning of this helpful revelation.  What about Redux, that wonderful front-end state management system that is separated into actions and reducers, separate from the React Components.  Do I need to create a new `klassActions.js` and `klassReducer.js` for adding, editing, and deleting klasses from my front-end Redux store?  Nope!  When creating my new React Native client folder, I can simply copy over my `/actions` and `/reducers` folders.  

Suddenly it feels like I'm not creating much from scratch except the actual React Native components.  And even those I've designed once before!

## Recreating my login screen part 1 -- `<App />`
<img src="https://www.dropbox.com/s/8j1ww0m5hu88kji/login.png?raw=1" alt="login screen" width="400px" style="margin: 0 auto; display: block"/>

Okay, so I have my blueprint.  There are three main things happening here: a NavBar, a Login component, and navigation to other "pages" on my app. 

My `<App />` component is where the magic starts.  First, we need to implement [React Navigation](https://reactnavigation.org/docs/getting-started), definitely one of the best documentations I've read to date.  React Navigation has a few options for how to create Navigation on a mobile device.  I'm using the Stack Navigator, which creates a stack of screens that you can navigate to.  You know how Gmail for your mobile device has that back arrow 'Inbox' on the top left of the screen, and when you click it, the screen slides over?  I'm pretty sure that's built using a Stack Navigator.  

<img src="https://media.giphy.com/media/aOften89vRbG/giphy.gif" style="display: block; margin: 0 auto;" width="100px" />

We also need to wrap our entire application in a Redux store, using the familiar `<Provider />` wrapper and my identical `store` from my ReactJS application.  

Back to the Stack Navigator. There are lots of ways to [customize the header](https://reactnavigation.org/docs/headers), and I wanted it to look just like my ReactJS NavBar.  So I initially set the backgroundColor to my baby blue rgb code.  

Next, I needed a starting screen to render my login component.  The `<Stack.Screen>` below is where this happens.  Other than sending it to my `<HomeScreen />` component, I also further configure the header.  When a user isn't logged in, the header only displays the logo, a login, and a signup link.  That's why I needed to configure it here, rather than in the top-level `<Stack.Navigator>` options.  Here, I use `headerLeft` to render my FlexSeats title logo.

My `<App />` code:
```
const App = () => {

  return (
    <NavigationContainer>
      <Provider store={store}>
        <Stack.Navigator
          screenOptions={{
            headerStyle: {
              backgroundColor: 'rgb(125, 166, 200)',
            }
          }}
        >
          <Stack.Screen
            name="Home"
            component={HomeScreen}
            options={({ navigation, route }) => ({
              title: false,
              headerLeft: (props) => <NavBarTitle props={props} />
            })}
          />
          <Stack.Screen name="Klasses" component={KlassesScreen}
            options={({ navigation, route }) => ({
              title: false,
              headerLeft: (props) => <NavBarTitle props={props} />,
              headerRight: (props) => <Logout navigation={navigation} />,
              gestureEnabled: false
            })}
          />
        </Stack.Navigator>
      </Provider>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App
```

## Recreating my login screen part 2 -- `<HomeScreen />`
I decided to make my `<HomeScreen />` component a helpful toggler between the `<Login/>` and `<Signup/>` components.  I use the React Hook `useState()` to create a boolean `login` that tracks whether or not `<HomeScreen />` should display the login or signup.  

Here I want interactivity with the header once again.  When we click the 'login' or 'signup' links, I'll `setLogin()` to true or false.  In order to [interact with the header](https://reactnavigation.org/docs/header-buttons) from within a screen, I had to use `React.useLayoutEffect()`.  I separated the actual rendering of the links into a component called `<NavBarRightLoggedOut/>`.  

My `<HomeScreen/>` code:
```
...
const HomeScreen = ({ navigation, getCurrentUser, currentUser }) => {

  const [ login, setLogin ] = useState(true)

  React.useLayoutEffect(() => {
    navigation.setOptions({
      headerRight: () => (
        <NavBarRightLoggedOut navigation={navigation} setLogin={setLogin} />
      ),
    });
  }, [navigation, setLogin]);

  return (
    <View style={styles.containerStyle}>
      {login ? <Login navigation={navigation} /> : <Signup navigation={navigation} />}
    </View>
  )
...
```

## Recreating my login screen part 3 -- `<Login />`
This was probably the most fun part, as I got to think through HTML/CSS styling and how things differ with React Native primitive components and JS styling.  

First off, we have an outer `<View>` functioning as an outer `<div>` wrapper.  Here's the styling:
```
  signupWrapper: {
    backgroundColor: "rgb(166, 152, 143)",
    alignItems: "center",
    justifyContent: "space-around",
    width: 300,
    height: 200,
    alignSelf: "center",
    borderColor: "#3e4444",
    borderWidth: 3,
    marginBottom: 130
  },
```

What's not present here is the styling on the parent element `<HomeScreen>`, which uses `justifyContent: "center", flex: 1`.  The flex styling makes `<HomeScreen>` fit exactly 100% of the height of the screen, and justifying the content centers `<Login />` horizontally.  

> In React Native, EVERYTHING is a flex box, and the flex direction defaults to vertical, while ReactJS defaults to horizontal.
> 
<img src="https://media.giphy.com/media/ui1hpJSyBDWlG/giphy.gif" style="display: block; margin: 0 auto;" width="250px" />

So in my `signupWrapper` styling, I align the items inside, which centers the login, text inputs, and buttons horizontally.  I justifyContent here as well, which puts an equal vertical space between these elements.  `alignSelf` lets me center the wrapper horizontally, and my massive `marginBottom` scoots it up a bit.

Using the `<TextInput/>` primitive element is pretty similiar to an `<input type="text" />` in HTML.  

```
<TextInput
        style={styles.textInput}
        autoCapitalize="none"
        autoCorrect={false}
        placeholder="Enter email"
        value={email}
        onChangeText={(newValue) => setEmail(newValue)}
      />
```

A few features are helpful and unique to iOS functionality, like turning off `autoCapitalize` and `autoCorrect`.  The slight change of `onChange` to `onChangeText` is semantic, but functions the same.  

Text inputs initially don't style at all, so I attached a style object:
```
textInput: {
    backgroundColor: "white",
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 5,
    width: 200,
    fontSize: 16
  }
```

Surprisingly, the biggest challenge here was mimicking my submit button!  In my ReactJS button, I was able to apply a background transparency easily.  But many of these CSS features don't existly natively on React Native.  So I had to import a package, and spend a good 20 lines on this one button.  But I definitely didn't want to leave that button code in the `<Login />` component, AND I wanted to be able to reuse it throughout my application!   So here is what it ended up looking like:

```
<BigButton
  callbackFunction={() => submitHandler()}
  title="Log In"
/>
```

I absolutely love this part of React: the reusing of versatile components.  Whether you grab them from the internet or make them yourself, they really compartmentalize things.  I won't post the actual 20-line code here, since at that point this blog post would be more code than writing!  But refactoring out that `<BigButton />` code into a reuseable component felt like a big learning moment for me.  

Finally, notice the callback function I send in to the BigButton.  This connects with that Redux logic, and calls the action creator, all things I had done already in ReactJS!

How's my component looking?  

<img src="https://www.dropbox.com/s/wkca1w692xfiozg/login2.png?raw=1" alt="new login screen" width="300px" style="display: block; margin: 0 auto;" />

Here's my final `<Login />` code:

```
import React, { useState } from 'react'
import { connect } from 'react-redux'
import { login } from '../../actions/currentUserActions.js'
import { View, Text, TextInput, StyleSheet } from 'react-native'
import BigButton from '../buttons/BigButton'

const Login = ({ login, navigation }) => {
  const [ email, setEmail ] = useState('')
  const [ password, setPassword ] = useState('')

  const submitHandler = () => {
    const data = {
      email: email,
      password: password
    }
    login(data, navigation)
  }

  return (
    <View style={styles.signupWrapper}>
      <Text style={styles.flexseatsTitle}>
        Login
      </Text>
      <TextInput
        style={styles.textInput}
        autoCapitalize="none"
        autoCorrect={false}
        placeholder="Enter email"
        value={email}
        onChangeText={(newValue) => setEmail(newValue)}
      />
      <TextInput
        style={styles.textInput}
        secureTextEntry={true}
        autoCapitalize="none"
        autoCorrect={false}
        placeholder="Enter password"
        value={password}
        onChangeText={(newValue) => setPassword(newValue)}
      />
      <BigButton
        callbackFunction={() => submitHandler()}
        title="Log In"
      />
    </View>
  )
}

const styles = StyleSheet.create({
  signupWrapper: {
    backgroundColor: "rgb(166, 152, 143)",
    alignItems: "center",
    justifyContent: "space-around",
    width: 300,
    height: 200,
    alignSelf: "center",
    borderColor: "#3e4444",
    borderWidth: 3,
    marginBottom: 130
  },
  flexseatsTitle: {
    fontSize: 24,
    fontWeight: "500"
  },
  textInput: {
    backgroundColor: "white",
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 5,
    width: 200,
    fontSize: 16
  }
})

function mapDispatchToProps(dispatch){
  return {
    login: (credentials, navigation) => dispatch(login(credentials, navigation))
  }
}

export default connect(null, mapDispatchToProps)(Login)
```

## Conclusion
Creating a separate React Native front-end has been a great way for me to expand my skills as a full-stack developer.  It gave me a better understanding of separation of concerns, helping me see why we separate out a back-end and separate out state management.  It also gave me clear visual pages to recreate, which forced me to learn more about React Native primitive components and styling.  It encouraged me to dive headfirst into React Native documentation, particularly the React Navigator.  And finally, it gave me small DRY wins when separating my code into smaller and more helpful components.

If you're a coder beginning React Native, why not try recreating one of your ReactJS apps on that mobile front-end?  

As always, happy coding!


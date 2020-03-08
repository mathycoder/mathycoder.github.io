---
layout: post
title:      "My First Whitboarding Tech Interview"
date:       2020-03-08 23:30:31 +0000
permalink:  my_first_whitboarding_tech_interview
---


I recently had my first experience whiteboarding.  Whiteboarding--or solving a coding challenge live without a computer--is certainly intimidating.  It's held a sort of mythical level of intimidation for me ever since I learned this would be part of finding a job as a software engineer.  I've heard the horror stories of Google interviewees failing miserably and being blacklisted from ever interviewing with them again.  I've pictured all of my knowledge emptying out of my brain and being outed as a fraud.  And considering that most actual programming is done on the internet with access to documentation and StackOverflow, part of me was a little frustrated that whiteboarding is a part of a technical interview at all.

<img src="https://media.giphy.com/media/hyMFaxhuQkZTq/giphy.gif" style="margin: 0 auto; display: block;" />

So when I grabbed that whiteboard marker and stood up at the whiteboard, I was understandably nervous.  But by the end of it, a lot of misconceptions I had held about whiteboarding were cleared up, and I actually enjoyed the process.  I also realized whiteboarding is very similar to the problem solving heuristic we created for our middle school students solving novel math problems -- Understand It, Plan It, Do It, Review It.  

## Understanding the Problem

<img src="https://www.dropbox.com/s/tj88dxhgxi00jop/whiteboard1.png?raw=1" width="400px"  />

When my technical interviewer presented the problem to me, I knew immediately I would need some clarification.  For one, I was nervous and wasn't fully grasping everything he was saying.  For another, the problem was stated verbally with few visual aids.  So instead of projecting a level of intelligence, I decided to be as open as possible about my understanding at every step in the process.  **My first takeaway: ask questions to clarify the problem, no matter how you think it makes you look.** 

My interviewer was more than happy to clarify aspects of the problem.  He offered to give me an analogy in any programming language I was most comfortable with.  Eventually, I came to understand the problem like so:

> Create a function that can determine whether or not a list of connected nodes is repeating or not.  For example: imagine instances of a class in Ruby or Javascript.  Each node has a 'next' attribute that points to the next node.  After sending in the first node as an argument to this function, determine if the nodes end up going in a circle, like the image above.  The function should return true or false.
> 

When I asked him what language he wanted me to code my answer, he said to pick any language I wanted, but that it could also be done in pseudo-code.  This leads to my **second takeaway: whiteboarding is less about language syntax and more about your process.**  If we're able to use pseudo-code, which I imagine is just writing what you'd like your code to do, then we really just need to remember the fundamentals rather than the specifics of the language we're using.  

Okay, now that I had clarified the problem and made sense of it, I moved on to next step in the problem-solving process.

## Planning a Strategy
Something I learned from my teaching experience is what NOT to do when coming to the board and solving a problem for the class.  I had many students who would come to the board, think silently, write for two minutes, and then sit back down.  So I definitely stood up at the whiteboard ready to talk about everything.  Basically, I embraced my inner Hermoine.

<img src="https://media.giphy.com/media/Wom8RnZcrZF9C/giphy.gif" style="margin: 0 auto; display: block;" width="400px" />

**Takeaway number 3: Talk through your thinking, especially what's giving you pause.**  The interviewer cares less about you getting the problem right and more about your thinking.  

I initially wanted to use a recursive function, where I could send in the next node when necessary.  As the function recursively moves through the next nodes, I needed some way to keep track of which nodes we'd seen.  So I said I could use an array to track this and look for any repeats.  But I was confused on where to keep this array, because I thought the function I was creating needed to be self-contained.  

This made me nervous.  "My plan isn't perfect!" I thought, and I didn't really want to start solving a problem until I knew my solution path accounted for everything.  But of course, that's not how it works when I really code, so why would that be the case here?  So I took a deep breath and jumped into my solution.  

## Just Do It
After lots of erasing and talking to myself, here's the solution I arrived at:

```
const list = []
const repeating = (node) => {
    list.push(node)
    const repeats = list.filter(n => node.next === n)
    if (repeats.length > 0) { 
		    return true 
	  } else if (node.next === null) {
		    return false
	  } else {
		    repeating(node.next)
	  }
}
```

First, I pushed each new node to the list array.  Then I used `filter()` to see if the next node was equal to any of the existing nodes in the list.  By checking the length of that `filter` return value, I made some conditional choices.  

This initial function didn't come totally smoothly, and was definitely as crooked on the board as my students would remember.  At one point I talked through being unsure of how to know if the list was over.  My reviewer gave me the hint that maybe its 'next' property would be `null`.  So I had to erase and rework my conditional logic.  More erasing.  More crooked writing.

At the planning stage, I didn't think through each of these steps.  All I had was a general outline of what I wanted the function to look like.  But just like when coding at home, most of the solution only comes once we start typing and experimenting.  Which leads to **Takeaway number 4: At some point, just dive in and start coding.**  If I had waited until I had thought out every step, I'd still be standing at that whiteboard.  Instead, now I had what felt like a working solution!

## Review It
Now that I had this solution, how did I know that it was right?  Well, I decided to talk through some examples.  First, I switched marker colors (definitely a teacher move) to show a concrete example.  I used the visual they provided as an example, talking through how 1 led to 2, and how our array was populating `[1, 2, 3...]`.  When I got to the logic of the 2 repeating, we saw that the code would return true.  Success!  I think that's worth a **takeaway number 5: test your solution with distinct examples.**

But that's only the first step I'd encourage my students to do when reviewing their answer.  Yes, make sure it works for some examples, but then the most critical question comes next: *is it the most efficient answer possible?*

My reviewer prompted me here to think this through using Big O notation.  I'm not a pro at thinking through solution efficiency this way, but I didn't really need to be to think this through.  After some thinking, I realized this solution was O(n^2).  That's because in a worst case scenario, it would need to run through each node AND during each function run it would need to `filter()` through each node again in my `list` array.  That's some quadratic growth right there.

Here I initially got really excited, because I thought I had a nice JavaScript trick that would cut out the inefficiency of my filter method: instead of using `.filter()`, what if I used `.indexOf()`?  I very excitedly reworked my problem, feeling like I remembered a function that would save the day.  Except, it didn't.  

<img src="https://media.giphy.com/media/bOQjg8lreAWD6/giphy.gif" style="margin: 0 auto; display: block;" width="400px" />

My interviewer said that under the hood, `.indexOf()` is still doing a linear search through each element in the array, which would keep my solution efficiency at O(n^2).  But then he offered up a suggestion: could I use an object/hash?

YES!  This seemed better, because I would be able to simply look up a key and see if it's in the object, rather than searching through each element of an array.  So I set to work revising:

```
const list = { }
const repeating = (node) => {
    list[node.id] = 1
    if (list[node.next.id]) { 
		    return true 
	  } else if (node.next === null) {
		    return false
	  } else {
		    repeating(node.next)
	  }
}
```

Now the solution was at O(n).  We good!  

## Final Takeaways
My interviewer was incredibly nice, and gave me feedback at the end.  First, he said he appreciated that even though I didn't have a perfect solution planned out, I jumped into writing.  He said many candidates get stuck and give up and simply put their markers down.

Second, he commented on my ability to take feedback.  Looking back at the process, this was not just me talking to myself.  This was a collaboration between me and the interviewer -- clarifying the problem, talking through hangups, looking for more efficient solution paths.  So for **a final takeaway: a whiteboard challenge tests your ability to collaborate.**  Can you take ongoing feedback, and consider ideas from others?  Do you stay positive and keep things moving when something gets tricky? 

For a lot of reasons, I loved this whiteboard interview.  It made me see how connected whiteboarding is to the problem-solving process I taught my students for years.  It helped me dispell myths about whiteboarding being intimidating, rather than a conduit for the joy of problem solving, communicating, and collaborating that coding can be.  And finally, it taught me how **inefficient arrays are for sorting**.  Never again, arrays.  Never again.  

I hope you found this helpful.  As always, happy coding!

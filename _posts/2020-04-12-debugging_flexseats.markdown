---
layout: post
title:      "Debugging FlexSeats"
date:       2020-04-12 21:04:23 +0000
permalink:  debugging_flexseats
---

For the past three weeks, I've been creating a React Native front-end to my FlexSeats web application.  It's been fun learning the ins and outs of React Native as I try recreating my ReactJS front-end in Native.  

While most of my blog posts lately have been about exploring new technologies and incorporating them into my applications, a huge chunk of my time is spent doing something else: debugging.  And while this normally happens behind the scenes, I thought it would be interesting to take a problem I literally stumbled upon a minute ago and try blogging about my debugging process.  

## The current problem
<img src="https://www.dropbox.com/s/upwtgov75cz3hmo/error%20gif.gif?raw=1" width="500px" style="display:block;margin: 0 auto;" />

The gif above shows what's currently wrong with my application.  This sample class starts off with 10 students.  When I open the 'Generate Pairs' menu and generate new seats, two students disappear.  When I check the student index, I see that two students are no longer even listed in my class anymore.  

Okay, let's get cracking!  There are plenty of places to look to determine what the error might be.

## Database check
My first thought is to load up my Heroku database and see if the students who disappeared are still even listed as students.  Perhaps they're getting deleted from the database when I generate new seats?!  In the gif, 'Chairys' and 'Amara' disappear from the seating chart, so let's see if they're still in the database on Heroku.

<img src="https://www.dropbox.com/s/c5npv906tx8sdkb/database-check.png?raw=1" width="800px" style="display:block;margin: 0 auto;" />

My backend is a Rails API.  I connect to heroku using the Heroku CLI, and then I make a few ActiveRecord queries to access my Postgres database.  I see that both of the students are indeed still in the database.  They both have a `klass_id` of 1, which after a quick check is indeed this current klass.  

Now I'm thinking the error might be that multiple students have the same assigned seat, and perhaps the error is in my sorting method.  Amara has a `seat_pair` of index 7, which means he should be in the 8th seat. But it appears that John is in that seat as well!  Similarly, Chairys has a `seat_pair` index of 1, which means she should be in the 2nd seat.  That's where David ends up.  And finally, I'm noticing that their seat indices haven't changed from their original seats.  

As a test, I decide to reload the page.  When I do so, I see that we still only have 8 students showing up, but now Chairys is in David's seat.  

**Conclusion thus far: when I generate new seats, some students are being assigned the exact same seats.  This creates some funky behavior, that makes it seem like students are being deleted from the seating chart and class.  All of this is happening on the back-end.  **

## Deeper and deeper we go
It's my belief that the error is now happening on the back-end sorting algorithm.  So let's go check that out and continue debugging in Rails.  

To do this though I'm going to need to run things locally rather than through Heroku.  I'm going to first set up the same situation and make sure the error happens locally on my machine.  Then I'll set up some `binding.pry` moments to debug my sorting algorithm.  

**Five minutes later...**

Awesome!  The error appears to be identical locally.  So let's look at this sorting algorithm:

<img src="https://www.dropbox.com/s/ht8qv4skg4fx242/sorting%20alg.png?raw=1" width="600px"  />
	
`group_by` in my test case was 'Academics', and this algorithm is for heterogenous pair sorting.  Line 30 uses a helper function called `sorted_students` to bring back an array of the current students ordered by their `pair_seat` index.  Let's throw in a `binding.pry` to make sure this is functioning properly.

<img src="https://www.dropbox.com/s/hxes13034bch3sl/pry1.png?raw=1" width="350px" />
	
Notice the `academic_score` attributes seem to be increasing and in numerical order.  So it seems like this method is working.  

Next I'll throw a `binding.pry` after the while-loop.  This should give me back an array of ten students as well, which it does.

Line 36 is interesting.  I wanted students to be paired up heterogenously, but for the desk pairs to  be randomly placed in the seating chart.  So my helper method `shuffled_pairs` should return ten students once again.  

<img src="https://www.dropbox.com/s/il2d85psqeu2eu2/pry2.png?raw=1" width="500px" />

Oh ho!  I think we've narrowed down where the problem is happening.  Let's take a look at this helper function.

```
def shuffled_pairs(sorted)
  binding.pry
  pairs_array = paired_array(sorted)
  final_array = []
  shuffled_pair_indicies = (0..(sorted.length / 2).ceil - 1).to_a.shuffle
  shuffled_pair_indicies.each do |index|
    final_array.push(pairs_array[index])
  end
  final_array.flatten
end
```

Okay, so `sorted` contains my 10 students in an array when I check it with a `pry`.  `paired_array` should return an array of 5 pairs, like this: [ [student1, student2], [student3, student4] ... ].  

What it actually returned was something bizarre.  `[ [ ], [student1, student2], [student3, student4], ... [student9]`.  Why was the first pair blank?  And why did the last pair only contain one student?  And so we go deeper.  Let's look at my helper method `paired_array`.

```
def paired_array(sorted)
  binding.pry
  pair_array = []
  pair = []
  sorted.each_with_index do |student, index|
    if index % 2 == 0 || index == sorted.length - 1
      pair_array.push(pair)
      pair = []
    end
    pair.push(student)
  end
  pair_array
end
```

Okay, so first I need to remind myself of how this method works.  I'm going to return `pair_array`, which should pair off each of my students.  `pair` starts as a blank array, and will hold individual pairs.  I then iterate through my `sorted` students array.  The logic inside the block says that when the index is an even number or if the index equals the last indexed student, we should push `pair` onto `pair_array`.

I'm noticing a flaw in the conditional logic.  Remember how bizarrely the first returned student pair was an empty array?  That's because the first index is 0, which technically returns true for `index % 2 == 0`, thus pushing the currently empty `pair` onto `pair_array`.  So let's adjust this logic to make sure the index is greater than 0.  

`if (index % 2 == 0 && index > 0) || index == sorted.length - 1`

But let's consider an extreme case.  If there's only one student, what would happen?  Well, the index would be zero, and we would want that student to be added to a `pair` before being pushed onto `pair_array`.  So maybe I need to rework this logic even more.  

Let's try this:

```
sorted.each_with_index do |student, index|
    pair.push(student)
    if index % 2 == 1 || index == sorted.length - 1
      pair_array.push(pair)
      pair = []
    end
  end
```

I've reversed the logic a bit.  First, we'll push the current student onto `pair`.  Then if the index is an odd number, we'll push `pair` onto `pair_array`.  This also works for the edge case of there being only one student.  I'm notice that it doesn't work for the edge case of there being zero students, so let's make a failsafe for that.  

```
def paired_array(sorted)
  binding.pry
  pair_array = []
  pair = []
  sorted.each_with_index do |student, index|
    pair.push(student)
    if index % 2 == 1 || index == sorted.length - 1
      pair_array.push(pair)
      pair = []
    end
  end
  pair_array = [[]] if sorted.length == 0
  pair_array
end
```

Okay, here goes nothing!  Let's see if this returns an array of 5 array student pairs!

<img src="https://www.dropbox.com/s/qkg54kengvu7yor/pry3.png?raw=1" width="300px" />

It does!  Okay, I think we're ready to test the algorithm in its entirety now.  I'll commit these changes to the GitHub repository, and then wait a few minutes to test it out on my React Native front-end.

<img src="https://www.dropbox.com/s/vha4c45zb8d8qch/success%20gif.gif?raw=1" width="500px" style="display:block;margin: 0 auto;" />

Woohoo!

## Conclusion
Debugging code gets more and more complex as the codebase gets larger.  And as I start moving toward my first software engineering position, I'm realizing the importance of test-driven development.  To debug this error, I had to explore three nested methods in my back-end sorting method.  Having tests for each one up front may have taken more time, but would have helped me zero in on the problem much more quickly in the long run.  

Also, I tested a lot of edge cases when thinking through the final method.  Yet my tests on my phone didn't actually test the edge cases.  I'd have to go back and check the sorting method on a new class with zero or one student.  Again, this builds the case for creating Ruby tests for each of my methods.  

I love discovering the need for things organically while working on my own projects.  Now I feel stronger than ever the desire to explore building an app using Test-Driven Development.  

And so ends my experiment of blogging through my debugging process.  It helped me really think through my code and process on a deeper level.  As always, happy coding!



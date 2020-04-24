---
layout: post
title:      "Algorithms: The Sliding Window Pattern"
date:       2020-04-19 18:28:25 -0400
permalink:  algorithms_the_sliding_window_pattern
---


Let's take a break from project-based blogs and take a look at one of the problem-solving patterns taught in the Udemy Course [JavaScript Algorithms and Data Structures Master Class](https://www.udemy.com/course/js-algorithms-and-data-structures-masterclass/).  Before going into some of the most common algorithms in Computer Science, he explores what he calls common Problem Solving Patterns that can help you write more efficient algorithms.  

## A connection between algorithms and algebraic reasoning
Algorithms in computer science remind me a lot of teaching algebraic reasoning to my middle schoolers.  A goal in algebraic thinking is to get students to *generalize* their strategies.  I remember giving my class a problem of the growing tables.  Each square table sat four students on each side, but when you put two tables together, you could only fit 6 students (2 on top, 2 on bottom, and one on each end).  Three tables pushed together fit 8 students (3 on top, 3 on the bottom, and one on each end).  And so on.  The question was how many students would fit if you pushed 100 tables together?

Students took many approaches to solve this problem.  Some drew pictures of each table combination.  Others thought to make a table, counting from 1 table up to 100 and adding 2 students each time.  But those that used algebraic reasoning where able to generalize the situation.  Some realized you could just do 100 + 100 (top and bottom) + 1 + 1 (the two ends).  Others generalized like this with pictures.  And finally, some generalized with an expression like 2T + 2.  

Algorithms feel a lot like these algebraic generalizations.  There's a way to solve the problem and get a function to do what you want, but inefficiently, like those students adding 4 + 2 + 2 + 2 + 2 + 2 + 2...etc.  Normally, this has a time efficiency of **O(n^2)** in Big O Notation.  Then there's something more elegant, some cool way to conceptualize the algorithm, that drops the time efficiency to **O(n)** or better.

With that connection, let's explore more computer science-y problem.  

## Solving a problem inefficiently

Here's a practice problem given in the Udemy course:

<img src="https://www.dropbox.com/s/c3wdmdubnfhq3bz/problem%201.png?raw=1" width="800px" />

Okay, let's first solve this problem like a student making a long table to get to table 100.  Imagine that this function is passed in `maxSubarraySum([100,200,300,400], 2)`.  So we need to find two consecutive numbers within this array with the largest sum.  

The first idea that comes to mind is to add each pair of numbers and return the highest sum.  Imagine starting with 100 and 200, and keeping track of their sum of 300.  Then move on to the next two numbers, 200 and 300.  Sum those as well, get 500, and now keep track of that sum.  Move on to the last two numbers, and get 300 and 400 summing to 700.  That's our new max sum, and we're done!  So we return that value of 700.  

This code might look something like this:
```
function maxSubarraySum(arr, num){
     let maxSum = -Infinity
		 for (let i = 0; i <= arr.length - num; i++){
		     let tempSum = 0
		     for (let j = i; j <= j + num; j++{
				     tempSum += arr[j]
				 }
				 if (tempSum > maxSum) { maxSum = tempSum}
		 }
    return maxSum
}
```

Now, this solution has a time efficiency of O(n^2).  That's because we're nesting two for-loops.  Imagine a giant initial `arr` and a large `num`.  We would have to loop through each number in the array, AND for each number we'd have to loop through all of the next `num` numbers.  We need a method that's the equivalent of 2T + 2, and we need it now!

## The Sliding Window Pattern
Here's how the instructor defines the Sliding Window Pattern:

<img src="https://www.dropbox.com/s/hpv6q04wjdad9qo/sliding%20window.png?raw=1" width="350px" />

Okay, so let's think this same problem through using the concept of a sliding window.

`maxSubarraySum([100,200,300,400], 2)`

So first we would need to add 100 + 200 and get our initial window.  But then instead of moving on to add 200 + 300, let's think about this.  Why add 200 again?  100 + 200 = 300, and that sum 300 already contains the 200.  Instead, to slide the window we need to add the next element and subtract the first element.  So we do 300 + 300 - 100 = 500.  

In this particular example, you may not see why this is so efficient.  So let's imagine a larger array:

`maxSubarraySum([100, 500, 300, 200, 400, 600, 250, 180, 300, 1000], 5)`

So the sum of the first 5 numbers is 100+500+300+200+400 = 1500.  To get the next sum, we COULD do 500+300+200+400+600, but let's just slide the window!  We take the sum 1500 and add the next number 600, then subtract the first number 100.  1500+600-100 = 2000.  As the width of the window gets bigger, the efficiency of the Sliding Window Pattern becomes more apparent.  We can always find the next sum just by adding the next number and subtracting the first, no matter if the window is 1,000,000,000 consecutive numbers.

Here's my solution in code:

<img src="https://www.dropbox.com/s/gcrzziin7e2jivb/solution1.png?raw=1" width="500px" />

The first for-loop sums up all the numbers in the original window, and sets that sum to `maxSum`.  Then, `tempSum` will check each of the next sums.  We do have a second for-loop, but it's not nested, and doesn't start from index 0.  Instead, it starts where the first for-loop left off.  For each element, we add it to our changing `tempSum` and subtract the first, which I got by doing `arr[j-width]`.  

Definitely having my 2T + 2 moment ;)

<img src="https://media.giphy.com/media/l2Je3qSgOVvFPdaNi/giphy.gif" style="display: block; margin: 0 auto"  width="300px" />

## A more challenging problem

<img src="https://www.dropbox.com/s/6t2zad6nd0twkd7/problem2.png?raw=1" width="800px" />

Okay, this one is notably different, and stretched my brain.  In the first problem, our sliding window was a set width, so it was easy to simply add the next and subtract the first elements to slide it forward.  Here, we are actually **looking for the shortest length** to meet a condition, which is a minimum sum.  

`minSubArrayLen([2,3,1,2,4,3], 7)`

The inefficient way to do this problem at first is the only one that really makes sense.  Let's do it without code:
* 2 + 3 is too small. 2 + 3 + 1... not there yet... 2 + 3 + 1 + 2 = 8, which is greater than 7.  So that was a length of 4.
* Go to the next element to start. 3 + 1 + 2 + 4 > 8, and again we needed 4 numbers.  So `minLength` is still 4.
* 1 + 2 + 4 gives us 7, and we only needed three numbers.  So `minLength` drops to 3.
* 2 + 4 + 3 also gives us `minLength` of 3.
* 4 + 3 gives us 7 with only two numbers, so our final return value is `minLength = 2`.

## Sliding that window
How could we apply the sliding window to this problem, even if the window doesn't have a set length?  Let's think through the logic first.

`minSubArrayLen([2,3,1,2,4,3], 7)`


Let's start by adding each consecutive element until we get a value greater than or equal to 7.
* **2,3,1,2**,4,3.
* currentLength = 4 numbers
* minLength = 4 numbers (the lowest so far)
* currentSum = 8

Here's where the window gets wonky.  Instead of going to the next element, let's subtract the first element (2) from our total.
* 2,**3,1,2**,4,3
* currentSum = 8 - 2 = 6
* currentLength = 4 - 1 number = 3 numbers
* minLength = 4 numbers (hasn't changed).

Okay, so those numbers don't sum to greater than 7, so now we slide the window to the right one spot.
* 2,**3,1,2,4**,3
* currentSum = 6 + 4 = 10
* currentLength = 3 + 1 = 4 numbers

Now let's chop of the front numbers while we can.
* 2,3,**1,2,4**,3
* currentSum = 10 - 3 = 7
* currentLength = 4 - 1 = 3 numbers
* minLength = 3!  We can switch it now!

And for the win, just with the bolding:
* 2,3,**1,2,4,3**
* 2,3,1,**2,4,3**
* 2,3,1,2,**4,3**

The last part helps the idea really click for me.  The current sum is 7 (1 + 2 + 4), and we add the last number (1 + 2 + 4 + 3).  Then we chop of the first number ( 1 + **2 + 4 + 3**) which still is greater than 7, so we do another chop (1 + 2 + **4 + 3**).  So we have this window that does slide up one at a time, but will also shrink the back of the window up multiple spaces if the window still meets the condition.  

Here's my code that follows that logic:

<img src="https://www.dropbox.com/s/iton9xmi7g6fhyk/solution2.png?raw=1" width="600px" />

The magic happens in the for-loop.  We keep adding the next element, but we also have a nested while-loop that will chop off the earlier numbers until the condition is met.  

While it may look at first glance like I've nested O(n) within O(n) to give us O(n^2), I don't believe this is how the code plays out.  The for-loop will go through each element of the array once.  And even though there's a nested while-loop, that will also only go through the whole array once **throughout the entire algorithm, not for each element**.  It can only chop off the front of the window once, and is essentially a loopy way of sliding the window forward.  

## Conclusion
The Sliding Window Pattern is one of a few basic algorithmic patterns the Udemy instructor explores before getting into things like Bubble Sort and other famous computer science algorithms.  As a lifelong teacher, I enjoy going through these problems like a student in my algebra middle school class.  It's fun to think through solutions, however inefficient they may be, because it makes the "Aha!" moment of discovering or learning about a more efficient strategy all the more meaningful.  

Thanks for reading, and as always, happy coding (and learning)!


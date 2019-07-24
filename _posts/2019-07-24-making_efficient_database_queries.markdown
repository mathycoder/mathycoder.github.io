---
layout: post
title:      "Making Efficient Database Queries"
date:       2019-07-24 18:35:28 +0000
permalink:  making_efficient_database_queries
---


![](https://www.dropbox.com/s/8qmdo5auzkzjn3h/gradebook.png?raw=1)


For my Rails Project: MyGradebook, I decided to create the gradebook I always wanted but could never find as a teacher.  I'd always wanted a gradebook better organized around the learning standards, with helpful data to inform pedagogical next steps.  Halfway through the project, I was happy with how it was looking and what it could do.  Teachers might really use this one day, I thought.  But then I started to notice how SLOW my main gradebook grade was loading.  I began to think how often the database might be called in an ever-expanding gradebook table.  What teacher would wait 5-10 seconds for a large gradebook to load?    

So here was another problem that came up while creating this project.  Coding is all about problem solving.  The problems tend to pop up unexpectedly, and then you're off brainstorming possible solutions.  For this particular problem, I decided to start with a concrete benchmark: the load time.  So I refreshed the main page and checked the loadtime in the terminal:

![](https://www.dropbox.com/s/ior8ohjzymh4n1s/runtime1.png?raw=1)

Okay, so this was my starting point.  I'd compare every change of code to this original loadtime.  To start looking for a piece of code to refactor, I decided to look at the individual SQL calls in the terminal to see if I could find any particularly repetitive lines of code.  Sure enough, I notice one line being called **15 times!** 

```
def klass_grades(klass)
  self.grades.select{|grade| klass.students.include?(grade.student)}
end
```

Looking at the SQL even more, I noticed that #select was getting called 7 times, one for each student.  This seemed like a time to try out the AREL methods I read about in the curriculum.  Something about looking through the grades, and then also looking through the associated students model at the same time seemed problematic.  So I read about the #includes AREL method, which preloads the needed association.  My new code looked like this:

```
self.grades.includes(:student).select{|grade| klass.students.include?(grade.student)}
```

This cut that one #select line of code from 7 calls down to 2! Okay, we were moving in the right direction.  But that still means the database was being called **10 times** for this one line of code.  What else was problematic about it?  The SQL calls in the database were titled "Student Exists."  Maybe that had something to do with my call to #include?  It seemed like each time we iterated through another grade, we were making a call to the database to see if students included the grade we were looking for.  

So this time I turned to the #where AREL method, and refactored the line to look like this:

```
self.grades.joins(student: :klasses).where("klass_id = ?", klass.id)
```

Instead of preloading with #includes, I simply wanted access to all of the tables connected with #joins.  And #where was much more useful than iterating with #select and calling #include? on the database each time.  #where returns every record that matches the query.  

Lo and behold, this one line of code was being called **1 time**, compared to a start of **15 times!**   That's when I went back and refreshed the page to compare the load time with my starting point:

![](https://www.dropbox.com/s/ks1pvaenmfj1618/runtime2.png?raw=1)

My loadtime had been cut in half!  It seemed like I had arrived at a decent method for improving the efficiency of my application.  I used this process to find other ineffeciencies throughout the application.  Sometimes I would find some line being called unnecessary, like this:

```
def average(klass)
   if !klass.grades.empty?
	   grades = klass.grades.where("student_id = ?, self.id).map{|grade| grade.score}.compact 
		 ....
```

It turns out the if-statement was completely unnecessary, as the #where method would provide me with the same information.  This prevented this method from being "double-called" for each average in computed.  

Other times my application would make dozens of calls whenever sorting was involved.  And my main insight was this: the more you have the database do the searching and sorting, the better!  For example:

```
def students_chronological_grades(student)
   self.grades.includes(:assignment).where("student_id = ?", student.id).order(date: :asc)
end
```

This method would have taken a dozen or more SQL calls to bring the same sorted information had I not used AREL's #where and #order.  

Throughout this problem-solving process, I was able to used Rails' AREL methods to cut down the load time to about 1/10 of the original load time.  Rails really makes it easy to think of coding as problem solving, because it has so many methods built in to serve as solutions, and a really helpful community of people answering questions.  On to the next problem!

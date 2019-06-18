---
layout: post
title:      "Learning to Refactor From Worked Examples"
date:       2019-06-18 16:43:58 +0000
permalink:  learning_to_refactor_from_worked_examples
---


As a former teacher, I'm always interested in both the best ways to teach and learn.  One of the methods for teaching math students new strategies without prescribing them is called "Worked Examples."  You show a student learning how to solve a proportion problem several different strategies written out.  Normally seeing these exemplars is enough for a student to think through the possibilities.  I always found it most useful after a student had used their own particularly laborious strategy to then see more efficient options.  See the coding parallel yet?

I've been trying to take advantage of all the ways Flatiron provides learning coding in my self-paced track, and perhaps the most fruitful one has been the worked examples.  After submitting a lab, students get access to a solution created by multiple participants.  I normally take a screenshot of their example and put it next to what I did:
![](https://www.dropbox.com/s/ha2t4r8fl5fkhns/Screen%20Shot%202019-06-18%20at%2012.20.57%20PM.png?raw=1)

With their exemplar on the left, and mine on the right, I get to comparing and googling.  My first learning opportunity happened when I compared my #genre_name method with theirs.

``` 
def genre_name
  self.genre ? self.genre.name : nil
end
```

I first tested the truthiness of self.genre, and used a ternary operator to keep the if/else statement to one line.  I thought this was as refactored as you could get.  But then I saw their code.

```
def genre_name
   self.try(:genre).try(:name)
end 
```

After consulting [APIdock](https://apidock.com/rails/v3.2.1/Object/try), I learned that the Ruby method #try won't raise a no method error if it that attribute doesn't exist, and instead will return nil.  No if-statement needed!


While this was cool, it didn't technically cut down any lines of code.  That's when I saw how inefficient my reader method #note_contents was compared their theirs:

My code:
```
  def note_contents
    all_notes = []
    self.notes.each do |note|
      all_notes << note.content
    end
    all_notes
  end
```

Their code:
```
  def note_contents
    self.notes.map(&:content)
  end
```

I had never seen the '&' sign in an enumerable before, so I googled that to arrive at this page, [How to Use the Ruby Map Method](https://www.rubyguides.com/2018/10/ruby-map-method/), where I learned that '&' tells the iterator not to take any arguments.  And #map simply returns an array of the return values in the block, which here just needed to be the string in notes.content.  

It's been illuminating as a teacher to be a learner again with coding.  I tend towards some of my students' habits, like using a painfully long strategy because they're more comfortable with it.  That's how I interpretted my use of #each here instead of #map.  And worked examples expose me to strategies I wouldn't have known otherwise, like using #map(&).  Here's to many more worked examples as I try and become a more elegant coder!




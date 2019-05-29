---
layout: post
title:      "Memory Lane -- My Sinatra Portfolio Project"
date:       2019-05-29 19:54:13 +0000
permalink:  memory_lane_--_my_sinatra_portfolio_project
---

# A lesson in organization

With my second project at Flatiron moving beyond a simple Command Line Interface, I knew I could start from a much bigger idea than my first project.  So I came up with the idea of creating an application for shared memories called Memory Lane.  Users could create a "lane" with others and begin to post "memories" there.  Below is an example of what these memories look like on the site when my project was ready to submit:

![](https://www.dropbox.com/s/wlvf4a584g77zv7/Screen%20Shot%202019-05-29%20at%203.19.22%20PM.png?raw=1)

What I didn't really anticipate going into the project was how much thought would need to go into the organization of the application before I could even begin coding.  I spent a few days on looseleaf and in [Gliffy](http://www.gliffy.com) trying to figure out the best way to organize the database tables.  I knew I wanted all users of a "shared lane" to be able to add memories and edit them.  And I wasn't comfortable going forward until I had a solid plan.

After a few days of really thinking through what my program would need to do, I came upon this structure:

![](https://www.dropbox.com/s/lvi6cxf8wxi4zmx/Screen%20Shot%202019-05-29%20at%203.11.16%20PM.png?raw=1)

I knew that I wanted users to have many lanes (I have my lane with my mom, another with my college friends), and also that lanes would need to have many users, so I created the join table user_lanes.  I also knew that I wanted one person to create a memory, but many people to edit the memory.  That's why I came up with the idea of recollections and photos.  Both tables belong to a given memory and to a given user, and there's no limit to how many recollections or photos a given user can add to a memory.  

Another lesson in organization came when I started fleshing out the home page, and all these new questions started to crop up.  What if someone in your lane posted an old memory from college?  How would your home page alert other users that there was a new memory if the memories are organized by the most recent?  

![](https://www.dropbox.com/s/o9znrdw0kacjnsw/Screen%20Shot%202019-05-29%20at%203.36.42%20PM.png?raw=1)

This is when I realized that it would be helpful to give each new or edited photo or recollection its own timestamp.  So I rolled back my database and added these attributes to the table.  I modified my controllers to set the timestamp to DateTime.now whenever adding or editing a recollection or photo.  From here, I needed an efficient way to find all of the recent images for a given user.  I decided the best way to do this would be in a Helper method in the application controller.  

    def all_images_sorted
      images = []
      @user.memories.each do |memory|
        memory.images.each do |image|
          images << image
        end
      end
      images = images.sort_by{|image| image.timestamp}.reverse
    end

    def recent_images
      images = all_images_sorted()
      images.length > 6? images[0..5] : images
    end
		
Instead of placing all of the code in one Helper method, I created one that would sort all of a user's images first, and then call that function when grabbing the recent images.  Being able to sort by the timestamp made this task much more straightforward.  

There's so much more I learned about organization--CSS responsive design and containers, systematic error testing, code refactoring and other types of helper methods.  I will definitely be taking all of these lessons with me to my next project.


		

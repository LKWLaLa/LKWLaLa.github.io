---
layout: post
title: Step by Step (finishing up my Rails project)
---

After a month and a half spent with Rails, I have finally begun refactoring and wrapping up my assessment project.  With it's high level "magic" helper methods, Rails is complex in every sense of the word.  In the process of learning I have found myself both excited by all of its possibility, and nostalgically missing the explicit ways of Sinatra.  My understanding of database relationships has been stretched to the breaking point.  Step by step (and error by error), I'm learning!

I thought it might be a good exercise to rewrite my Sinatra project using the Rails framework.  That way I could look back at my own work and compare the two approaches.  Doing so was beneficial, however, the many-to-many relationships that were easily set up in my first project, grew into a dark labyrinth of broken associations when implementing nested attributes, forms, and resources.  

My project, called "Step-O," is an organizer for dance steps and the Youtube videos they may be found in.  Steps and videos have a many-to-many relationship, through a join table "timemarkers."  ("Timestamps" is already a macro in ActiveRecord, and I didn't want to duplicate the name.)  The timemarkers join table contains columns for the foreign keys step_id and video_id, as well as an additional column to store the actual timemarker field (the timestamp at which a step occurs in a video).  Oh boy, did I have a heck of a time setting this up.  

In order to write to the additional field on the join table, I needed to create a nested form - two layers deep.  Originally, I nested steps inside of a video, and then timemarkers inside of steps.  So what I had was something that looked like this:  Video > Steps > Timemarkers.  (Or Step > Videos > Timemarkers).  This however, does not work.  I may need to write an additional blog when I am better able to articulate this correctly, but my takeaway was this:

A nested form can only create objects through a one-to-many or one-to-one relationship.  To say that nesting steps inside of videos (or vice versa) is creating objects through a many-to-many relationship is deceptive.  It might appear that way on the surface, but in actuality, we cannot associate the two without writing to a row on the Timemarkers join table.  So the functional order of nesting should actually be this:  Video (singular) > Timemarkers > Step (singular) or Step(singular) > Timemarkers > Video (singular).  A Video can have many Timemarkers (or the same video_id can be present on many rows of the join table), while a single row on the Timemarkers join table (because we will iterate through our individual Timemarkers rows before creating further nested associations) will associate with only one Step (or step_id).  

[This stackoverflow post](http://stackoverflow.com/questions/13506735/rails-has-many-through-nested-form) helped me immensely.  

In addition to grappling with database associations, working on this project also taught me more about Devise, OmniAuth, error messages, AREL queries and how to filter my index page using scope methods and a dropdown menu, as well as dynamically embedding Youtube videos - lots of fun!  Honestly, there is so much more I could write about, but it will have to wait for another blog post. 

On to more learning!





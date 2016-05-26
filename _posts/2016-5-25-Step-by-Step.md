---
layout: post
title: Step by Step (finishing up my Rails project)
---

After a month and a half spent with Rails, I have finally begun refactoring and wrapping up my assessment project.  With it's high level "magic" helper methods, Rails is complex in every sense of the word.  In the process of learning I have found myself both excited by all of its possibility, and nostalgically missing the explicit ways of Sinatra.  My understanding of database relationships has been stretched to the breaking point.  Step by step (and error by error), I'm learning!

I thought it might be a good exercise to rewrite my Sinatra project using the Rails framework.  That way I could look back at my own work and compare the two approaches.  Doing so was beneficial, however, the many-to-many relationships that were easily set up in my first project, grew into a dark labyrinth of broken associations when implementing nested attributes, forms, and resources.  

My project, called "Step-O," is an organizer for dance steps and the Youtube videos they may be found in.  Steps and videos have a many-to-many relationship, through a join table "timemarkers."  ("Timestamps" is already a macro in ActiveRecord, and I didn't want to duplicate the name.)  The timemarkers join table contains columns for the foreign keys step_id and video_id, as well as an additional column to store the actual timemarker field (the timestamp at which a step occurs in a video).  Oh boy, did I have a heck of a time setting this up.  

In order to write to the additional field on the join table, I needed to create a nested form that was two layers deep.  Originally, I nested steps inside of a video, and then timemarkers inside of steps.  So what I had was something that looked like this:  Video > Steps > Timemarkers.  (Or Step > Videos > Timemarkers).  

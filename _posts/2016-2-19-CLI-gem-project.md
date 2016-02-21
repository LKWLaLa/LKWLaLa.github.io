---
layout: post
title: A Gem of a Project
---

Today I have enthusiastically finished my first Ruby project, and am currently writing my first blog post (ever).  Ringing in the new year with new digital experiences.  :-)

As a student in [Flatiron School's Learn program] (https://learn.co), this project (a published Ruby gem), is to be my first progress assessment.  And I have to say, although my gem is still clunky, I'm feeling proud to have built something that works (mostly) as intended - from scratch!  

My first challenge was the process of getting started.  A blank slate can be both exciting and intimidating, and I felt both. Not to mention, I wanted to make something useful, as opposed to simply fulfilling the project requirements, so I was hung up for a few days on **what** to make. In many lectures, our instructor Avi has said, "Build the code you wish you had."  So that is what I attempted to do. 

I've wrestled with severe food allergies my whole life (nuts and shellfish), and if my social plans for the evening potentially include going out to eat, this more than likely means I will spend hours beforehand scouring restaurant menus, in an attempt to find a place with a low occurrence of allergens.  Well, I really wish I had a tool to do that!  So I attempted to build one...  My Ruby gem, "eatable," currently scrapes and iterates through menus on menupages, searching for regex terms I have specified in two constants: NUTS and SHELL_FISH.  I've also specified a maximum threshold I will allow for each list (only two or less items on the menu may contain nuts, and three or less may contain shellfish).  The program will return the name/address/phone number of restaurants that meet those requirements.  And generally - I'm excited to say - it works! 

Well, mostly.  

Problem #1.  It takes quite a long time to request and return data from each individual menu.  The menus have their own pages, so to sift through 400 restaurants in Soho, NYC, I currently need to make 400 http requests with nokogiri.  Perhaps there is a way to optimize this?  Which brings me to the next issue...

Problem #2.  They're on to me!  Too many requests from my IP address, and menupages shuts me out.  (Fortunately, only temporarily at this point.)  To deal with this, as well as problem #1, I've narrowed my search down to only the first 20 restaurants listed in a particular neighborhood, but it makes the program much less useful.  I'd love to be able to search an entire neighborhood for safe food options, in one go.









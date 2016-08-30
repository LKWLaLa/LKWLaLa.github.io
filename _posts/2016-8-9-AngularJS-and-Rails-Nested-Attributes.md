---
layout: post
title: AngularJS and Rails Nested Attributes
---

Today I began adding the finishing touches to my AngularJS project.  A few styling changes left, refactoring, and then much studying before the final assessment.  Exciting!

A lesson I have learned repeatedly over the past few months is the importance and challenge of good design.  Planning ahead, drawing things out, and anticipating object relationships (desired behaviors as well as potential pitfalls) will make for a happier development experience.  

Design patterns in Angular, with its asynchronous nature and powerful views require a different frame of reference from the conventions of Rails.  This caused some initial headache in determining how my API would function - particularly in working with nested attributes.  I wanted to mimic the behavior of a Rails nested form in Angular, but how? I could still use  `accept_nested_attributes_for`  on the backend, but with no place now for its  `<%fields_for%>` partner.  In place of erb were JSON and $http requests.  Luckily, with Rails and all its magic, we've been guided to look under the hood (or behind the curtain, shall we say) to understand the concepts on which it is built.  After a bit of research, the solution to my problem turned out to be fairly simple.  

To signal to Rails that I would like to write to two tables at once, I needed to send params in the exact format it would be looking for.  A quick detour to explain my data structure:  The app I've created (called StoryMâché) facilitates collaborative storytelling.  The top level fields I am interested in are a story's title, updated\_at, and total\_contributors (I increment this field when a story is updated).  A story has-many contributions, located on a separate table, for which I am keeping track of the body, author, and created\_at.  When a new contribution is created, I would like to handle this by *updating* the story, and simultateously writing the new contribution to the contributions table.  

Rather than sending an $http request that looks like this:

```javascript
this.updateStory = function (id, author, body) {
    return $http.patch(API_URL + '/stories/' + id, {
       contributions:[
       {author: author,
        body: body}
       ]
     });
    };
```


A few small modifications create the params Rails is looking for:

```javascript
this.updateStory = function (id, author, body) {
    return $http.patch(API_URL + '/stories/' + id, {
      story {
       contributions_attributes:[
        {author: author,
        body: body}
        ]
       }
     });
   };
```

If this seems a bit unweidly, Rails ActionController has another bit of "magic" that can optionally be added to the API controller, 
`wrap_parameters include: [:contributions_attributes]` 
which allows requests to be submitted without specifiying the root element:

```javascript
this.updateStory = function (id, author, body) {
    return $http.patch(API_URL + '/stories/' + id, {
       contributions_attributes:[
       {author: author,
        body: body}
       ]
     });
   };
```  

And with that, (as well as adding the `accepts_nested_attributes_for` in my Story model in Rails) my Angular service was able to send the appropriate params to the API, and voila - the story update action is writing to two tables!




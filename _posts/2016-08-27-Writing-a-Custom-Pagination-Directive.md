---
layout: post
title: Writing a Custom Pagination Directive in Angular
---

There are many resources out there to assist with pagination of an Angular app, one example being [Michael Bromley's pagination directive](https://github.com/michaelbromley/angularUtils/tree/master/src/directives/pagination), a nice "plug-n-play" solution for divvying up your index items. While normally one would not re-invent the wheel unnecessarily, I decided to implement my own pagination in my latest project for the sake of learning.  Below is my take away from the experience.

##Applying both pagination and filtering

In my first attempt at implementing pagination, I added logic into my controller which would slice my index page data (a collection of stories) and return only my designated number of stories per page.  Considering the current page number, the beginning and ending index numbers in the slice would change accordingly, like so:

```javascript
ctrl.paginate = function () {
  var begin = ((ctrl.currentPageNo - 1) * ctrl.itemsPerPage);
  var end = begin + ctrl.itemsPerPage;
  ctrl.displayedItems = ctrl.allItems.slice(begin,end);
};
```   
Clicking a button or link in the view would call one of the following methods to update the page number:

```javascript
ctrl.previousPage = function () {
  ctrl.currentPageNo--;
  ctrl.paginate();
}

ctrl.nextPage = function() {
  ctrl.currentPageNo++;
  ctrl.paginate();
}
```  

I simply needed to execute `ctrl.paginate()` in my controller, so that the first pagination would occur as soon as my controller was initialized for that view.  

Now at first glance, that seemed to produce the behavior I was looking for.  I had decided there would a maximum of 12 stories per page, and when I clicked the "next" button, the next set of 12 stories appeared.  Super cool.  This was working.  Then I attempted to type something into my search input...

It only searched/filtered the 12 items in the current view.    



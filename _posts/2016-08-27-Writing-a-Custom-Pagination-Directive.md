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



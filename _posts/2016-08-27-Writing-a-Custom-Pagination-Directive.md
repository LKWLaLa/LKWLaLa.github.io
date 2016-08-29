---
layout: post
title: Writing a Custom Pagination Directive in Angular
---

There are many resources out there to assist with pagination of an Angular app, one example being [Michael Bromley's pagination directive](https://github.com/michaelbromley/angularUtils/tree/master/src/directives/pagination), a nice "plug-n-play" solution for divvying up your index items. While normally one would not re-invent the wheel unnecessarily, I decided to implement my own pagination in my latest project for the sake of learning.  Below is my step through and take away from the experience.

##Applying both pagination and filtering

####The basic pagination logic:

In my first attempt at implementing pagination, I added controller logic which would slice the index page data (a collection of stories) and return only the designated number of stories per page.  Depending on the current page number, the beginning and ending index numbers in the slice would change accordingly, like so:

```javascript
ctrl.paginate = function () {
  var begin = ((ctrl.currentPageNo - 1) * ctrl.itemsPerPage);
  var end = begin + ctrl.itemsPerPage;
  ctrl.displayedItems = ctrl.allItems.slice(begin,end);
};
```   
Clicking a button or link in the view would call one of the following methods to update the page number and then re-paginate:

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

I'd execute `ctrl.paginate()` in my controller, so the first pagination would occur as soon as my controller was initialized for that view.  The `ctrl.displayedItems` value would be passed into `ng-repeat` to add only those items to the DOM.

####Filtering:

Now at first glance, that seemed to produce the behavior I was looking for.  I had decided there would be a maximum of 12 stories per page, and when I clicked the "next" button, the next set of 12 stories appeared.  Super cool.  This was working.  Then I attempted to type something into my search input...

It only searched/filtered the 12 items in the current view.  Ugh...of course.  And in addition to the search input, I was also implementing an orderBy drop down selector, as well as another custom filter - all of which would no longer work correctly.  All filters were presently added in the standard way to the `ng-repeat` element, so the only items I had access to were the 12 `ctrl.displayedItems` passed into `ng-repeat`.  

The solution?  Apply the filters **first** and **then** paginate.   

##Filtering from within the controller

In order to pass pre-filtered, pre-paginated data to the `ng-repeat` element, I moved my filtering into the controller.  This was achieved by using Angular's $filter API, which can be injected as a controller dependency.  It works like so:

```javascript
function myCtrl($scope, $filter)
{
    $filter('filtername')(arg1,arg2);
}
```

Because my particular index page implements many different filters, I researched how to chain them together, and ended up with the following method:

```javascript
//default value includes all stories
ctrl.filteredStories = stories.data; 

ctrl.refilter = function () {
  filtered = $filter('filter')(stories.data, ctrl.search);
  //apply the second filter to the result of the first
  filtered = $filter('greaterThan')(filtered, ctrl.minContributions); 
  //apply the third filter to the result of the second
  filtered = $filter('selectedOrder')(filtered, ctrl.sortBy);
  //now we have a filtered collection that can be paginated
  ctrl.filteredStories = filtered;
}  
```

From the view, `refilter()` gets called using the `ng-change` directive whenever one of the input boxes triggers the $digest cycle.
For example:

```html
<!-- Note: I am using controller as syntax here-->
 <input ng-model="ctrl.search" 
        ng-change="ctrl.refilter()"                
        placeholder="Search">
</input>  
```

Then `ctrl.filteredStories` (in the code example above this one) becomes the starting point / value passed in for pagination. 

```javascript
ctrl.displayedItems = ctrl.filteredStories.slice(begin,end);
```


##Resetting to page #1

With filtering moved into the controller and the result of that filtering passed into the pagination method, the more desired value of `ctrl.displayedItems` can now be passed into the `ng-repeat`:

```html
<ul ng-repeat="story in ctrl.displayedStories">   
    <li>
      <a ui-sref="story({ id: story.id })">
        {{story.title}}
      </a>
    </li>
<ul> 
```

At that point in my logic, I realized there was still one more problem.  When a user types into the search input (or any filter), the pagination should begin at page one, regardless of the page the user is currently on.  If a user is on page 3, searches for stories about 'dogs', what if there are only 2 dog stories?  

To remedy this, it is worth noting I also wrote a resetPageData() method which I called at the end of refilter(), however ultimately this was implemented in a different way when I reached the final stage of this process: refactoring ALL of the above logic into a custom directive.

## Moving all of this into a 
#custom pagination directive










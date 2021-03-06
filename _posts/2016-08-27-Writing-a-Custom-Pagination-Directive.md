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

I'd execute `ctrl.paginate()` in my controller, so the first pagination would occur as soon as my controller was initialized for that view.  The `ctrl.displayedItems` value (from the first code example) would be passed into `ng-repeat` to add only those items to the DOM.

```html
<ul ng-repeat="story in ctrl.displayedItems">   
    <li>
      <a ui-sref="story({ id: story.id })">{{story.title}}</a>
    </li>
<ul> 
```

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
<ul ng-repeat="story in ctrl.displayedItems">   
    <li>
      <a ui-sref="story({ id: story.id })">{{story.title}}</a>
    </li>
<ul> 
```

At that point in my logic, I realized there was still one more problem.  When a user types into the search input (or any filter), the pagination should begin at page 1, regardless of the page the user is currently on.  If a user is on page 3, searches for stories about 'dogs', what if there are only 2 dog stories?  

To remedy this, it is worth noting I also wrote a resetPageData() method which I called at the end of refilter(), however I won't go into too much detail just yet.  Ultimately the page reset is achieved in different way at the final stage of this process: refactoring ALL of the above logic into a custom directive.

# Refactoring into a custom directive

So, after going through the steps above, I finally had working pagination!  The only problem left was that of separation of concerns.  The little voice in my head kept nagging; the controller shouldn't be responsible for all of this.  Pagination....  that feels like presentation logic.  It should be handled in the view, and ideally made into a reusable component.  What if I want pagination on other pages as well?  After more research, it seemed the best option would be refactoring this into a custom directive.

## The pagination directive element

I began by restricting my directive to "E" for an element.  

```javascript
angular
    .module('app')
    .directive('myPagination', MyPagination);

function MyPagination() {
  return {
      templateUrl: 'directives/page-buttons.html',
      restrict:  'E'
      }
``` 

When added to a view, the pagination directive element will render html for the next page and previous page buttons.  That is accessed by the templateUrl property, and the actual button html is located in my file 'directives/page-buttons.html', which looks like this:

```html
    <div ng-if="currentPageNo < totalPages">
      <a ng-click="nextPage()">Next >></a>
    </div>

    <div ng-if="currentPageNo > 1">
      <a ng-click="previousPage()"><< Back</a>
    </div>
```  

On my stories index view template, where I want the buttons to appear, I added my new pagination directive element:

```html
<my-pagination
      all-items="ctrl.filteredStories"
      current-items="ctrl.displayedItems"
      items-per-page=12
      ></my-pagination>

```

You'll notice `<my-pagination>` has three attributes:  `all-items`, `current-items`, and `items-per-page`.  Because I wanted the directive to be reusable, I chose to use an **isolate scope**.  This means the directive will not have access to the scope of any parent controller on the page, and therefore cannot directly read the value of `ctrl.filteredStories`, which is the collection to be paginated.  The above syntax creates a 2-way binding with the `ctrl.filteredStories` property in the stories controller's scope.  For more on isolate scopes in custom directives (and custom directives in general), check out Dan Wahlin's excellent lesson [here](http://weblogs.asp.net/dwahlin/creating-custom-angularjs-directives-part-2-isolate-scope).

I also added the additional property `ctrl.displayedItems` onto my original stories controller.  Like `ctrl.filteredStories`, this is 2-way bound to my pagination directive (see above). Although the pagination directive's controller will calculate the value of `ctrl.displayedItems`, this must still be accessible on the stories controller scope, so that it can be used with `ng-repeat` as demonstrated earlier.

The default value set in the stories controller is:

```javascript
ctrl.displayedStories = [];
```

*(Side note: Setting this default value to be an empty array actually prevents the page from flashing when it loads, as `ng-repeat` adds items to the DOM before Angular's other directives are linked to the html.  Meaning, the `ng-repeat` will execute and display whatever has been passed in before the pagination actually occurs.)*

Inside of my pagination directive file, I bind the the values to the directive's scope, and move the pagination logic from the stories controller into the pagination directive's own controller:

```javascript
function MyPagination() {
    return {
        templateUrl: 'directives/page-buttons.html',
        restrict:  'E',

        scope: {
          allItems: '=',
          currentItems: '=',
          itemsPerPage: '='
        },

        controller: function ($scope) {
          $scope.currentPageNo = 1

          $scope.previousPage = function () {
            $scope.currentPageNo--;
            $scope.paginate();
          }

          $scope.nextPage = function() {
            $scope.currentPageNo++;
            $scope.paginate();
          }
          
          $scope.paginate = function () {
            var begin = (($scope.currentPageNo - 1) * $scope.itemsPerPage);
            var end = begin + $scope.itemsPerPage;
            $scope.currentItems = $scope.allItems.slice(begin,end);
          };

          $scope.updateTotalPages = function(){
            $scope.totalPages = Math.ceil($scope.allItems.length/$scope.itemsPerPage);
          }

          
          $scope.paginate();
        }
    };
}
```

## Resetting the page after filtering

 Now, the only thing left to do is reset back to page 1 when filtering occurs.  To do this, I added a watcher:

 ```javascript
//if allItems changes (because a filter is run and
//the value of ctrl.filteredStories changes)
//pagination should reset

$scope.$watch('allItems', function() {
  $scope.currentPageNo = 1;
  $scope.updateTotalPages();
  $scope.paginate();
 });
 ``` 

 And the full pagination directive file then looks like this:

 ```javascript
angular
    .module('app')
    .directive('myPagination', MyPagination);

function MyPagination() {
  return {
      templateUrl: 'directives/page-buttons.html',
      restrict:  'E',

      scope: {
        allItems: '=',
        currentItems: '=',
        itemsPerPage: '='
      },

      controller: function ($scope) {
        $scope.currentPageNo = 1

        $scope.previousPage = function () {
          $scope.currentPageNo--;
          $scope.paginate();
        }

        $scope.nextPage = function() {
          $scope.currentPageNo++;
          $scope.paginate();
        }
        
        $scope.paginate = function () {
          var begin = (($scope.currentPageNo - 1) * $scope.itemsPerPage);
          var end = begin + $scope.itemsPerPage;
          $scope.currentItems = $scope.allItems.slice(begin,end);
        };

        $scope.updateTotalPages = function(){
          $scope.totalPages = Math.ceil($scope.allItems.length/$scope.itemsPerPage);
        }


        //if allItems changes (because a filter is run and
        //the value of ctrl.filteredStories changes)
        //pagination should reset

        $scope.$watch('allItems', function() {
          $scope.currentPageNo = 1;
          $scope.updateTotalPages();
          $scope.paginate();
        });
        
        $scope.paginate();
      }
    };
}
 ``` 
 And there we have it - hooray! A reusable custom pagination directive. 
















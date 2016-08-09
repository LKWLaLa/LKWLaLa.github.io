---
layout: post
title: AngularJS and Rails Nested Attributes
---

Today I began adding the finishing touches to my AngularJS project.  A few styling changes left, refactoring, and then much studying before the final assessment.  Exciting!

A lesson I have learned repeatedly over the past few months is the importance and challenge of good design.  Planning ahead, drawing things out, and anticipating object relationships (desired behaviors as well as potential pitfalls) will make for a happier development experience.  

Design patterns in Angular, with its asynchronous nature and powerful views require a different frame of reference from the conventions of Rails.  This caused some initial headache in determining how my API would function - particularly in working with nested attributes.  I wanted to mimic the behavior of a Rails nested form in Angular, but how? I could still use `accept_nested_attributes_for` on the backend, but with no place now for `<%fields_for%>`.
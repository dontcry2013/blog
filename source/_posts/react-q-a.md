---
title: React Q&A
date: 2021-04-19 10:14:42
categories: [Web]
tags: [React]
---
## React.PureComponent

The difference between React.PureComponent and React.Component is that `React.Component doesn’t implement shouldComponentUpdate()`, but React.PureComponent implements it with a shallow prop and state comparison.

<!--more-->

## Functional vs Class-Components in React

* Syntax
  * A functional component is just a plain JavaScript function which accepts props as an argument and returns a React element.
  * A class component requires you to extend from React.Component and create a render function which returns a React element.
* Lifecycle Hooks
  * You can now use the useEffect hook to use lifecycle events in your functional components.
* So why should I use functional components at all?
  1. Functional component are much easier to read and test because they are plain JavaScript functions without state or lifecycle-hooks
  2. You end up with less code
  3. They help you to use best practices.
  4. The React team mentioned that there may be a performance boost for functional component in future React versions.
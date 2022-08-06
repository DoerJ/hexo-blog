---
title: Part 1 - A Look into Angular from a High-Level Perspective
date: 2020-08-11
tags: web development, Angular, app design
index_img: /images/thumbnail/angular.png
---
### Scalability
When it comes to web development, the scalability is always one of the must-haves for a modern web app. Today people are more and more replying on the online busineses. The growing complexity of the business rules have driven the web app to be more scalable of expanding functionality modules. 

Furthermore, most of web apps today are Single Page Application(SPAs) in which all components look like residing in a single page, achieved by navigations. In old times, in order to visit different web page, user sends URL address that is suffixed with `.html` to request a new html file from backend. In a SPA, all components are pre-loaded during initilaization. That means, more responsibilities have be shifted from the server side to the client side. 

SPA is more responsive since there wouldn't be any HTTP request overhead for retrieving html files, and of course, more interactive. It's not hard to see the frontend logics becomes more and more complex. **The frontend today doesn't just display the user data. It processes user data.** Having only the view components is not enough, and that's why the modern frontend frameworks such as Angular brought up concepts of `model` and `service`. Therefore, a well-grounded frontend architecture is strongly needed to maintain the app modules, and manage user data flows. 

### Typescript in frontend development 
**Scalability** may have varying definitions for software engineer of different areas. The backend software engineers will probably think of scalability more in terms of the performances of the application, such as the capabilities of resolving network traffic spikes. The frontend enginners, on the other hand, interprets scalability more as the degrees in which the application can be expanded with new functionality modules.

To make the frontend app scalable, the developers need to follow certain programming paradigm that can structure app components in a matainable way. Object-Oriented Programming is definitely among those programming paradigms. Actually since ES6, it was not hard to see that JavaScript has started bringing in the concepts of OOP on top of the functional programming. And as mainstream frameworks such as `Redux` and `Angular` stepped in, OOP has been truely reinforced in frontend development.

The benefits that Typescript brings to the frontend devleopment is noticable. In Typescript, all app modules including `model`, `service`, and `component` are class-based, meaning more conveniences for developers to abstract out objects, implement interfaces, and inherit class properties. Hence in some degrees, frontend developers are enforced to follow with OOP paradigm with the help of Typescript, which is a good practice that keeps the project maintainable.

On top of the idea of OOP, Typescript also aligns with the architectural design of modern frontend frameworks, especially Angular. What really makes Angular standing out among other mainstream frameworks such as Vue and React, with respect to the ability of "engineerizing" the frontend project, is that Angular holds the philosophy of thinking application in layers and falling proper responsibilities into the proper layers. And Typescript has played an important role on supporting this philosophy. 

### Coming up in the next part
In the next part, we'll take a deep look at how Angular designs the application into different layers. What are those layers? What are their responsibilities? And how this certain architectural design pattern benefits the scalability of the application? These are the questions that will be answered in the next. Stay tuned!








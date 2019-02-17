---
layout: post
title:  "MVC pattern and Rails"
date:   2019-02-17 15:50:00 +0200
categories: [tech, architecture, rails]
---

A few years ago, a friend of mine from GitLab asked me a question.
Now this question is used as one of the main during the screening process.
There are my thoughts on it.

### Question

> Being a pretty standard Rails application, GitLab is built using the MVC design pattern.
Please describe in as much detail as you think is appropriate what the responsibilities of the Model, View, and Controller are,
both in general and in Rails specifically, and what the benefits of this separation are.

### Abstract

While I was thinking about the answer I am just flashing back to the words:

> _Managing complexity is the most important technical topic in software development. In my view, it is so important that Software’s Primary Technical Imperative has to be managing complexity._
>
> &mdash; **_Steve McConnell. Code Complete_**

Keep it in mind let's say that MVC pattern is used to separate application's concerns. This pattern separates the modeling of a domain (M = Model), a presentation (V = View), and actions based on user input (C = Controller). It leads to a modular design. And this separation of concerns allows to reduce complexity of the system.

### Model

Model, in its true sense, is responsible for a data and a business logic. The business logic should be encapsulated in the model, along with any implementation logic for persisting a state of the application. Ideally, the model object should not have explicit connection to the view objects that represent its data and allow user to edit that data — it should not be concerned with the presentation layer.

#### In Rails

ActiveRecord binds together business objects and database tables to create a persistable model. Models are Ruby classes that are closely coupled with the database. Models deal with data validations, associations, transactions, etc.

### View

Views act as the presentation layer. They are responsible for representing of content. There should be minimal logic within views, and any logic in them should be related to content presentation. A major purpose of view objects is to display data from the application’s model objects and to enable the editing of that data. View objects are should be decoupled from model objects.

#### In Rails

ActionView is responsible for handling view template lookup and rendering. Views should absorb a data from Controller in most cases. View's logic, if any, can be extracted into Helpers (additional level of modularity).

### Controller

Controller is the initial entry point, and it is responsible for handling user input, selecting which model types to work with and in which way to represent a response. In other words, it controls how the app responds to a given request. Controller objects can also perform setup and coordinating tasks for an application and manage the life cycles of other objects.

#### In Rails

ActionController is responsible for making sense of a request and producing an appropriate response (often through a view rendering). Typically, it is concerned with communicating with the database and performing CRUD actions where necessary. Provides access to the request object, parameters, cookies, sessions, response.

### Benefits

> _Models + Views + Controllers = **Modular System**_

This is one of the possible approaches to manage complexity.

I believe a healthy complexity management leads to a healthy development process, maintainability and supportability.


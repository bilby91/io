---
layout: post
title: "Rails to Hanami"
description: "Simple Hanami JSON API from someone with a Rails background"
tags: [hanami, rails]
published: false
---

A couple of month ago I discovered a small community of ruby developers that where building a new web
framework, [Hanami](http://hanamirb.org). Since then, I've been following pretty closely the progress of
the project. In the last version. Hanami integrated with [ROM](http://rom-rb.org) in their data layer, so,
I decided it was a good idea to play a little bit with it and create a simple REST API to see how it worked out.

# What we are going to build ?

We are going to build a REST JSON API for a classic TODO application. The app while include the following.

## Endpoints

### Lists

- GET - Fetch list of lists (/lists)
- GET - Fetch list (/lists/:id)
- POST - Create list (/lists)

### Tasks

- GET - Fetch tasks (/lists/:list_id/tasks)
- GET - Fetch task  (/lists/:list_id/tasks/:id)
- POST - Create task (/lists/:list_id/tasks)

## Entities

## List

- id (Integer)
- name (String)

## Task

- id (Integer)
- description (String)
- list_id (Integer)

# Development

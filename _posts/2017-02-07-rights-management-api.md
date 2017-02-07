---
layout: post
title: User and Permissions Management Service
image:
tags: [Elixir, Phoenix, Service]
---

# Project Description

Rights management is a core piece of many applications. And it certainly seems
like one of those things that has been done before. Or not?

The world is not the same, but user and permissions management has little
changed over the year. Most software either does a custom implementation or
they resort to LDAP or ActiveDirectory.

These traditional services are database driven monstrosities and are not
very handy for newer web based applications. Furthermore, it is very
difficult to do anything 'modern' with them. Modern things:

  * Collaboration
  * Monitoring in real time
  * Undo
  * Audit trail
  * Behavior tracking
  * Hooks for integrations
  * Simple management of Organizations, Teams, Users, and Projects (Resources)

# Collaboration

This may sound strange because most permission management systems work
in a sort of 'god' mode. Changes are made, you press refresh and see the
changes.

But User management is inherently a collaborative effort. Why not take the
logical step and make it a true collaborative application? By using and
tracking a single application state using a command-event architecture
you know the exact state of the entire system at any change point.

Most collaborative applications fail because they are implemented with
traditional technologies, such as a typical Java Enterprise or Rails project.

The trick here is to always have a live cache, backed by a persistent
database. Reads don't hit the database. Writes do. Everything change is
serialized. See the last section (Implementation) to see how we can do this
with Elixir/Phoenix.

# Proposal

Develop an integratable user and permissions management service. One that
supports a dual-mode interface: both HTTP and notifications over web-sockets.

When integrating the software can use a simple REST API and simply get the
results. However, an application interested in tracking the changes can plug
into the websockets channels and see things as they happen in real time.

Using a command-event architecture would guarantee serialization of changes
and yield a very responsive system. Characteristics of these systems are that
there might be 100 reads for every one write. So using a cached backed process
would give high performance.

A command-event architecture would also allow playback and undo. The aggregate
state being the application of the events in the order they were received.

Audit trails naturally come directly from the command-event stream.

Monitoring in real time is facilitated by the dual mode API. The advantage
being that many people could be monitoring in real time the changes made to the
system without adversely affecting performance.

Behavior hooks such as miss-use could be tracked by observing the command-event
queue and applying analytics to the usage.

Integration hooks could be implemented - need to look at how this could be done.
The idea is that any actual authentication is done externally to the service via
hooks.

# MVP

The Minimum Viable Product would manage simple structure:

* Organizations
* Teams
* Users
* Resource (typically Project)

There's a lot of joins here, but this is how I've modelled it:

Users: Can be member of Team. Can be member of Project. Can be member of Organization.
Can be an admin member of Organization.

Team: Can be member of Project. Is a member of an Organization.

Organization: just a name really.

Project/Resource: Can be owned by User or Organization.

If a User is a member of an Organization, then the User can create a Project.

An Admin User of an Organization can create Teams and add Users to Teams.
A User doesn't have to be a member of an Organization, but if they are not,
then they can't create Projects.

# Implementation

If you are interested in this project send me an email, and we can talk about
the architecture. I have all the drawings and am working on an Elixir implementation.

Elixir/Phoenix provides some great tools to implement a command-event and
process backed cache, as well as web sockets. I've been researching and developing
collaborative applications for years and understand the challenges of creating
scalable and robust applications with guaranteed consistent views for all users.

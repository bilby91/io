---
layout: post
title: "Open Source contributions at Suttna"
description: "Summary of some open source contributions at Suttna for the BotBuilder ecosystem"
categories: [open-source, node, typescript]
published: true
---

Ten months ago we started a new project with [@santiago.doldan](https://github.com/santiagodoldan), we called it [Suttna](https://suttna.com). The idea was to build a bot capable of scheduling standups in multiple chat clients. This side project turned out to be a real business and quite a few users started using it 😊

In order to build a cross platform bot, we needed a piece of technology that would abstract the communication layer between the different chat providers. Similarly, we needed a common set of constructs that would allow us to define user conversations. We found Microsoft's BotBuilder framework to be a really good option. It solved both problems by decoupleing networking communication with chat providers using `Connectors` and modeling user conversations using a very powerfull `Dialogs` API.

We did some initial testing around the framework and decided to give it a try!

During our journey developing [Suttna](https://suttna.com), we have built multiple libraries to solve different problems in the BotBuilder ecosystem.

This are some of the libraries we have developed and open sourced this far.

Some clarifications:

- Botbuilder -> Microsoft open source botbuilder framework for node.js and C#

- BotFramework -> The Microsoft Bot Framework provides just what you need to build and connect intelligent bots that interact naturally wherever your users are talking, from text/SMS to Skype, Slack, Office 365 mail and other popular services

## [botbuilder-slack](https://github.com/suttna/botbuilder-slack)

We found multiple issues while using BotFramework's channels.

Most important issue is that you don't "own" the OAuth process. BotFramework handles OAuth and then sends an event to your bot. Main issue with this setup is that information about the installer is not available in the event.

Missing events. Slack sent some interesting 
events that we needed but BotFramework was not 
forwarding (probably because they were using the RT 
API in some cases). First solution for this problem was [botbuilder-slack-extension](https://github.com/suttna/botbuilder-slack-extension). It basically added a new connector to listen and emit the missing events using Slack's webhooks.

Commands. Another feature that we wanted to use but was not avaiable using Botbuilder were Slack commands.

To solve all this issues and some other stuff, we decided to build a custom Slack connector that was 100% compatible with Botbuilder's ChatConnector. This would mean that we clould swap connectors without chaning any extra code and it should "just work".

So far, Suttna has been running botbuilder-slack without any issues in production for more than 4 months.

Source available here https://github.com/suttna/botbuilder-slack
 
## [botbuilder-redis-storage](https://github.com/suttna/botbuilder-redis-storage)

When users communicate with Suttna using BotBuilder, user's conversation state needs to be stored somewhere. BotBuilder provides an in-memory implementation that is fine for testing and maybe development as well but definitely not reliable for a production environment. We looked for alternatives and we couldn't find one that solved our constraints. We decided to build a small storage library that saves conversation's state in `redis`.

This library implements BotBuilder's `IBotStorage` interface, plain and simple.

Source available here https://github.com/suttna/botbuilder-redis-storage

## [botbuilder-markdown](https://github.com/suttna/botbuilder-markdown)

Botframework attemps to standarize the message's format but we found multiple issues trying to maintain a set of translations that should work in Slack and Microsoft Teams. We had several issues with new lines and code formatting to name a few. 

We decided to write a middleware that would translate valid markdown text into each platform's markdown flavor.

At the moment the botbuilder-markdown supports Slack and Microsoft Teams.

Source available here https://github.com/suttna/botbuilder-redis-storage

# Conclusion

It has been really interesting to work with Botbuilder so far. There is a new experimental version of Botbuilder being developed. Anyone interested in contributing with idea can follow the development here https://github.com/Microsoft/botbuilder-js.

Have you tried any of our libraries ? Do you find them useful ? 

Wanna know you thoughts and any feedback!

Enjoy 🎉
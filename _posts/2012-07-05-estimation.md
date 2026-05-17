---
title: "Estimation"
date: 2012-07-05 09:29:53 
author: shajeebhameed
layout: page
slug: estimation
status: publish
---

We, as programmers, are constantly being asked 'How long will it take'? And you know, the situation is almost always like this: 

  * The requirements are unclear. Nobody has done an in depth analysis of all the implications.
  * The new feature will probably break some assumptions you made in your code and you start thinking immediately of all the things you might have to refactor.
  * You have other things to do from past assignments and you will have to come up with an estimate that takes that other work into account.
  * The 'done' definition is probably unclear: When will it be done? 'Done' as in just finished coding it, or 'done' as in "the users are using it"?
  * No matter how conscious you are of all these things, sometimes your "programmer's pride" makes you give/accept shorter times than you originally suppose it might take. Specially when you feel the pressure of deadlines and management expectations.

Many of these are organizational or cultural issues that are not simple and easy to solve, but in the end the reality is that you are being asked for an estimate and they expect you to give a reasonable answer. It's part of your job. You cannot simply say: I don't know. As a result, I always end up giving estimates that I later realize I cannot fulfill. It has happened countless of times, and I always promise it won't happen again. But it does. What is your personal process for deciding and delivering an estimate? What techniques have you found useful? \------------------------------------------------------------------------------------------------------------ Software estimation is the most difficult single task in software engineering- a close second being requirements elicitation. There are a lot of tactics for creating them, all based on getting good requirements first. But when your back's against the wall and they refuse to give you better details, Fake It: 
  1. Take a good look at the requirements you have.
  2. Make assumptions to fill in the gaps based on your best guess of what they want
  3. **Write down _all_ your assumptions**
  4. Make them sit down, read, and agree to your assumptions (or, if you're lucky, get them to give in and give you real requirements).
  5. Now you have detailed requirements that you can estimate from.

\-------------------------------------------------------------------------------------------------------- It depends on the organization and how the estimates are used. If the estimate is just to provide a general idea on when it will be ready, I can generally do a quick estimate based on my experience. Often times I will include any uncertainty or possible variations with the estimate along with how the changes may impact other areas of the system and the extent of regression testing required. If the estimate is used for anything contractual or in a scenario where more precise timing is required, I do a full work break down. This is more work and requires more in depth thinking about the design and changes to the system, but is much more accurate, especially for larger pieces of work. In either case, on-going communication is key. If you do run into something unexpected, make it known at the time instead of waiting until the deadline. If the requirements are not-clear, make sure you document your understanding of them and the functionality that you plan to deliver. This is also helpful with any assumptions you make. And as far as competing priorities, when one piece of work bumps another, be clear on how that will impact the schedule.

---
layout: post
title: An Introduction to testing
excerpt: "Who, What and Why when it comes to Software Testing?"
modified: 2015-06-29
tags: [agile,software development,testing,team]
comments: true
published: true
image:
  feature: intrototesting/testing.jpg
---

## Testing. Such good times.
So, i've recently been asked to talk to some trainee software engineers about testing, the guys running the course they are on are doing an excellent job covering the hows of testing, so they asked me to give a bit of an idiots guide (i.e. something I understand) to the WHY of testing.

## Video!
<iframe width="560" height="315" src="https://www.youtube.com/embed/ywBg_8ptz8g" frameborder="0" allowfullscreen></iframe>

## Download the Presentation
[Direct Download of the Powerpoint](https://www.dropbox.com/s/jpl4wlvh8t0u4ph/testing-intro.pptx?dl=0)
<br/><br/>
If you feel like delivering this yourself for some reason, go right ahead. Drop me a line if you want help.

![Creative Comons - Attribution](../images/cc-attribution.png)

[Creative Commons - Attribution](https://creativecommons.org/licenses/by/2.0/uk/)

## Putting it together
This is the outline of my thought process for putting together a presentation on the subject.

See below for the outline of the talk before it existed...

### 1. Testing
![Testing](../images/intrototesting/1-testing.jpg)
<br/>
<br/>
The art of making an automated list of your failures. Every presentation needs some jump off point...

### 2. Why Bother?
![Whats the point](../images/intrototesting/2-why.jpg)
<br/>
<br/>
So, an automated list of failures, that doesn't sound the best. Surely we could be spending time doing something else...(want to go towards the idea of NOT doing testing)

### 3. Where's my stuff?
![Build all the things](../images/intrototesting/3-stuff.jpg)
<br/>
<br/>
...like building stuff. So much stuff. Angry customer guy wants all the stuff. Gimme it. Now. (this is something people may face in real life)

### 4. Stuff, Ca$H, Not Terrible
![Constraints](../images/intrototesting/4-triangle.jpg)
<br/>
<br/>
The golden triangle of making things. Given that often we aren't doing things for free, and most projects have a budget, the Ca$h part is usually immovable, so you have a choice between the other two, lots of stuff, or stuff that is not terrible. (some context around why this sort of thing might happen)

### 5. Pick One
![Make A Choice](../images/intrototesting/5-pick.jpg)
<br/>
<br/>
Yup, you get to pick one. So, lets go for the lots of stuff for our money model. (So decision time, what if, we don't bother testing)

### 6. Lets test that theory
![No Tests?](../images/intrototesting/6-planes.jpg)
<br/>
<br/>
So, cool, Planes, they have software, what if we stick loads of stuff on there, spend all our DogeCoins on features...(extreme case to illustrate point)

### 7. Uh...
![Not a good plan](../images/intrototesting/7-no-tests.jpg)
<br/>
<br/>
Maybe not. Not sure about you, but personally, i'll take a plane that gets there in one piece, over a plane that has un-tested automatic ground avoidance with built in Kanye West tracking. (it might go horribly wrong)

### 8. What sort of tests
![Test Whut?](../images/intrototesting/8-test-pyramid.jpg)
<br/>
<br/>
So, the guys will no doubt go over this, you go from fine grained tests all the way up to pretty loose manual exploratory testing. You need this for a lot of reasons. Automated testing so you can deploy continuously...but that is a whole other topic. (little mention of the testing approach we adopt, the guys will cover in way more detail, but worth adding context for the next idea)

### 9. The other Pyramid
![Proove What?](../images/intrototesting/9-pyramid-2.jpg)
<br/>
<br/>
So, there is another test pyramid. The test pyramid of WHAT it is you are trying to prove. (From [Fifty Quick Ideas to Improve Your Tests](https://leanpub.com/50quickideas-tests))

### 10. Who does this?
![Who?](../images/intrototesting/10-who-writes.jpg)
<br/>
<br/>
So who is actually responsible for this, the old view was you have a "tester" that does this stuff. (Now lets pose a question...see if there are any replies)

### 11. Not sure this is a good plan
![Prepare the Cannon!](../images/intrototesting/11-cannon.jpg)
<br/>
<br/>
Maybe though, that having the people that write the code, blast it at someone who didn't to write tests for it isn't the best idea in the world. (Lets show the bad plan, talk about the reasons why this isn't cool)

### 12. EVERYONE DOES IT
![GO TEAM!](../images/intrototesting/12-everyone.jpg)
<br/>
<br/>
Yes, thats right, it is everyone on the teams responsibility for testing. Fair enough developers may write more unit tests than others, possibly someone with more expertise might write more tests for workflow / UI. But, the role is shared. (Little diversion in responsibilities)

### 13. Build the right thing
On an Agile project, the team are responsible for the design, development and deployment of functionality. Testing is part of that. Ultimately we are trying to make sure that what we are doing is building the right thing. So it goes beyond just the Mike Cohn test pyramid, it goes back to the Pyramid of Ultimate Utility (Make the final point that this is ultimately about building the right thing for users)
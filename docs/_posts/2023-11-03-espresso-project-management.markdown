---
layout: post
title:  The Espresso Framework
date:   2023-11-03 18:48:44 +0000
---

As a solo venture I've tried most project management tools. Agile, Trello lists, paper lists, GitHub Projects, JFDI, etc. All mis-fired for me, I was planning but not doing.

I need a way to manage projects, that doesn't suck the joy out of making progress.

Unable to find something ready-made, I decided to create the Espresso framework. 
Something simple, designed for solos, managed in the time it takes to drink an espresso.

### Solo Differs

When "team work" features are stripped out, project management is objectively very simple.

But in a solo venture we need less objectivity, and rather more subjectivity. 
Our own motivation and time is far more important than labels, charts, and overly-planning the future.

Motivation and time are precious ingredients for a solo. 
The way we work creates and consumes both. 
The Espresso framework inherently boosts motivation and protects time. 
Without them we falter. Combined, they make us unstoppable.

Because Espresso values motivation and time, it increases our chance of success. 
However individual success is defined, as a solo it must incorporate daily self-determination. 
Such as the freedom to go for a coffee, or taking some time to recharge.

Solos must continually find the energy to endure, as well as everything else.

### How It Works

I chose to use <a href="https://far-oeuf.com/2023/11/02/we-were-promised-jetpacks.html" target="_blank"> the abstract idea of a journey</a> to model my lists and tasks.
A destination represents a significant chunk of work, and waypoints are tasks that to get me there.

```
journeys:
  - destination: significant chunk of work
    waypoints:
      - name: a task that gets me closer
        done: MMYY (on completion)
        component: name of component (optional)
        note: | (optional)
          brief descriptive text why this task is necessary
      - name: another task that gets me closer
        note: | (optional)
          brief descriptive text why this task is necessary
```

I choose to store this in YAML format (<a href="https://far-oeuf.com/assets/example_journey.yml" target="_blank">fuller example</a>), because it lets me automate some things later on.
Other formats would work too.

The usual 'team' accoutrements (Due Date, Owner, Meetings, etc) are not required.
Keep it simple, save effort.

Maintaining this data file is the only admin required.
I tend to do this on the iPhone over an espresso in a cafe.
Updating the _done_ element of the current waypoint automatically rolls the next one into view.

#### Focus

Focus is required to get things done well.

The path to a destination is rarely linear.
Therefore waypoints can have a _component_ element, calling out detours.

Thus tasks from many sources can be funnelled into a single list of waypoints.
Solo projects have many elements, but only one pair of hands.
One list at a time, with one task at a time.

The daily face of the Espresso framework is its focus view.
And, because it's automated, it comes for free whenever I update the YAML data file.

Focus view shows the destination, current waypoint, and a little look back at progress made.
A motivation boosting nudge, instead of facing large ToDo lists every day.

<img src="/assets/espressoshellmessage.png" alt="Screenshot espresso framework focused tasks on linux shell login" width="100%" height="auto">

#### Language Matters

It took me a while to realise that objective language can be demotivating.
In a team environment objectivity leads to clearer communication, so the cost is worth it.

But we don't talk to ourselves objectively, I hope.
I use language that motivates, that speaks to my strengths.

I rewrote this task:

```
Store API data on filesystem
```

to:

```
Simplest way to store Strava data on filesystem
```

As an engineer I am nudged by the challenge of finding the simplest way to do something.
Simple is always better than complicated.

Other examples of motivation nudging language:

```
Decide...
Learn...
Figure out...
```

All of these challenge me to overcome, rather than merely do work.
It's a small thing, but it seems to matter.

#### Automated Boosting

In a solo venture, something is unfinished if it costs time to manage.
We are fortunate that there are so many ways to automate in computing.
The simplest works best.

Each time I update the project data file, automation ripples the change out to my work environment.

When I log into my computers they automatically display the latest focus view (above).
It's another nudge to keep me right, keep me moving forward.

I’ve saved the niftiest thing till last.
I was able to automate the focus view as a wallpaper for my phone.

Every time I glance at my phone, the waypoint to focus on is right there.
If I’m having a moment, then I can see solid progress is being made and take heart. 
It’s my handheld motivation booster.

And by moving non-essential app icons off the home screen, I must swipe past my project in order to waste time. 
Another little nudge to spend effort wisely. 
I really wish I thought of this years ago.

All this happens while I enjoy my espresso.

<img src="/assets/iphonewithespresso.jpg" alt="Screenshots of iphone with espresso framework wallpaper" width="100%" height="auto">

---

-- started in various cafes in Orléans, France  
-- continued on the 11:00 train from London to Edinburgh  
-- continued in Detour cafe, Edinburgh  
-- refactored in Detour cafe, Edinburgh

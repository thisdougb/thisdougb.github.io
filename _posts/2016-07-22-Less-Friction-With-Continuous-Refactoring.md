---
layout: post
title: Less Friction With Continuous Refactoring
permalink: /devops/less-friction-with-continuous-refactoring
tags: ansible,role,roles,playbook,play,devops,engineering,cyclomatic,complexity,continuous,refactoring
---
One of the perils of Agile development is rooted in the belief that progress should be measured by change and newness.
Far be it for me to assume the persona of [Moros](http://greekmythology.wikia.com/wiki/Moros "Moros in Greek myth"), but I have a sense of déjà vu.
Back in the days of 'physical tin' we'd regularly find forgotten yet business-critical applications churning away on PCs under vacant desks.
The application owner had long since left the company, and with it any hope of identifying what the PC was actually doing or its login.
VMs and containers are even easier to leave behind.

DevOps and Agile don't really say much about *technical debt*, it's not cool to talk housekeeping.
Yet the amount of money (in time, energy, etc) wasted in looking after parallel legacy infrastructure, whether physical or virtual, can be surprising.
Bits of virtual infrastructure is being left active in the wake of rapid Agile projects, as they up-version their way into production.

Agility and progress can be more pragmatically measured not in your newest piece of infrastructure widget, but the age of your oldest.
What portion of your infrastructure is simply baggage from the past?
Can you visualise the wake left behind by your DevOps style, does it look proportional?

So, am I really saying our technology trailing-edge is where reality exists?
The technological span between your oldest supported infrastructure and your newest project is certainly a telling indicator.
The degree of friction that resists development progress is an unfathomable function of the length of the technology span in your environment.
And it's this degree of friction, albeit unmeasurable, that indicates how agile a team or a company are.

Technical debt is a recognised phenomenon in programming, particularly so in Agile development.
When you take the quick option instead of considering the best option, you tend to create additional work that will need to be addressed in the future.
It's no different for us in DevOps, rattling through versioned VMs on a project you can quickly lose track of what's what and end up accidentally supporting all of them.

Along with Continuous Integration and Continuous Deployment, I think we need to add Continuous Refactor to our workflows.
I recently wrote an iPhone app to try out continuous refactoring, and the process resulted in faster overall development and better final code.
My motion detecting camera app was only 400 lines of code, but I wrote the code fresh over multiple beta versions.
Rewriting methods and logic, with the benefits of hindsight, is one of the most effective ways to produce better code.
In the end I knew the logic so well that a re-write took only about an hour.

What has my iPhone app experiment to do with DevOps?
Well I was trying to minimise the technological span of the code.
When that span is narrow, everything in the code/design is relevant, intuitive and understandable.
Redundant variables and methods have been removed.
Variables who's meaning has changed have been renamed appropriately.
Some extended if...elseif... logic is perhaps rewritten as a better suited switch statement.

Importantly the code is more readable by someone else, and easier to understand when you come back months later.
The app design moved forward, and the trailing-edge moved forwards with it.
Real progress.

I want to develop the Continuous Refactor idea, and bring it into my DevOps work.
The benefits in software development teams are well known, and we can have those same gains when developing infrastructure as code.
Let's avoid situations where no-one wants to touch a piece of infrastructure because it's well out of date, like a mouldy piece of cheese at the back of the fridge.

I'm starting out small.
Below I parse a simple Ansible playbook, graph the tasks and then find all the possible paths through the plays.
This gives me a crude value for the complexity of the playbook.
A team-wide complexity threshold recognises that no-one enjoys hacking obscure code, and that we recognise its impact on team performance.
I incorporate this complexity threshold into our Jenkins continuous integration workflow, flagging up breaches for peer review.

```
$ python ansible_playbook_complexity.py
Loading playbook test/site.yml...
[node 0]: PLAY: test play, ROLE: common, TASK: common : update hostname, changed_when: None
[node 1]: PLAY: test play, ROLE: common, TASK: common : update /etc/hosts, changed_when: something is not true
[node 2]: PLAY: test play, ROLE: webtier, TASK: webtier : install apache, changed_when: s == 1
[node 3]: PLAY: test play, ROLE: webtier, TASK: webtier : ensure apache is running, changed_when: None

Built graph.....
{
    "0": [
        1,
        2
    ],
    "1": [
        2,
        3
    ],
    "2": [
        3
    ]
}

Found paths.....
path:  [0, 1, 2, 3]
path:  [0, 1, 3]
path:  [0, 2, 3]
```

Once we turn the code part of 'infrastructure as code' into data in its own right, we can analyse it and apply relevant rules and thresholds.
Do we have too many logic branches based on supported software version numbers?
Did our last commit increase complexity by 30%, if so automatically flag for peer review maybe?
There are lots of possibilities when you treat configuration as data, but the overall aim must be to keep the *friction caused by technological span* to a pragmatically low level.

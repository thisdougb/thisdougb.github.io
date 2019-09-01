---
layout: post
title: How To Choose DevOps Tooling
permalink: /ansible/how-to-choose-devops-tooling
tags: ansible,puppet,terraform,cto,devops,engineering
---

(State the surface problem, terraform v ansible)

(Identify the real problem, no discipline to choose tools on merit)

(Why this matters)

---

**(State the surface problem, terraform v ansible)**

Are technology people inherently irrational when faced with making choices?
I have pondered this for many years, with increasing frequency as the lifetime of software tools has rapidly decreased.
What is it that makes some of us so vehemently for and against software tools, and how can we become more effective at choosing software tools?

I have witnessed too many Ansible v Terraform v Puppet arguments, that I am pushed into defining a method to guide the choice.
In this post I will describe this method using automation tooling as the topic.

**(Identify the real problem, not discipline to choose tools on merit)**

I asked some artist friends of mine to recommend a sketch pad for ink drawings.
Immediately they fired a volley of questions at me about where, when and how I'd be sketching.
We summarised my goal in the user story:

```
I would like to sketch the people and places I see while travelling to DevOps clients,
as I often have short amounts of free time sitting in cafes, or train stations, etc.
```
What struck me was how familiar their line of questioning was, to software development.
They then outlined, distilling down till they were in agreement, the main requirements and constraints:

```
Sketchbooks for travelling must be low-hassle and robust (hardback, no greater than A5 size).
Type of pen should be available to buy in most medium sized towns.
Type of pen ink must be fast drying.
Pens must not leak ink.
Sketchbook paper should be of sufficient weight to hold the ink.
```

We were quite far into the conversation and yet no-one had even mentioned a brand name.
My experience of technology discussions is that from the outset, you already know what everyone is advocating (Terraform or Ansible, Ruby or Python, etc).

We (technology and DevOps people) seem to start with the answer, and work backwards to justify it.
This, I feel, is what DevOps tooling discussions are really about.
We have our favourite tool and we try to justify its use in every case we come across.
We often wilfully neglect the user story currently in play.

My artist friends closed off by removing from the set of possibilities any products that didn't meet the requirements.
They presented this:

```
Sketchbooks to consider are A and B, pens are X, Y, and Z.
The paper in sketchbook A works best with pen Y.
The paper in sketchbook B works bext with pen X.
Sketchbook B and pen Z is the cheapest combination.
```
Only at this point did any of them offer which of A, B, X, Y, and Z they actually prefered.
Only at this point did subjectivity come into it.
Any combination in this set meets my user requirements, so I am now free to pick and reject what I like on a subjective basis.

The real lesson from the art world is that tools are just tools.
To evangelise about a pen branded X, is to miss the point completely.
Likewise, to dismiss an individual tool, be it a pen or a piece of software, on subjective grounds is not only bad practice but also limiting your potential.

>Technology ("science of craft", from Greek τέχνη, techne, "art, skill, cunning of hand"; and -λογία, -logia[2]) is the collection of techniques, skills, methods, and processes used in the production of goods or services


When engineering in a DevOps environment, the broader _and_ deeper your toolbox, the better placed you are to pick the right tool for the job.
So let's take this approach to automation tooling.

**(offer the solution)**

When I start with a client the first job is to figure out what they really need to achieve, their DevOps user story.
When thinking about automation I split tasks into two categories, infrastructure tasks and operational tasks.
Infrastructure tasks are creating and destroying things, ALBs, instances, VPCs, etc.
Operational tasks are everything else.

With the client we fill out the _examples_ section of this table (typically these come from their run-books):

Tasks | Infrastructure | Operations
--- | --- | ---
Examples | Create VPC<br>Create subnets<br>Destroy RDS instance<br>Create/destroy ephemeral testing env's | Manage ephemeral testing env's (query, extend)<br>Modify ECS clusters<br>Database restore testing<br>Flip database cluster nodes for patching<br>Update ECS task definitions<br>Rotate AWS IAM deployment api keys

Companies where developers rule the roost, tend to only be able to list infrastructure tasks.
This is usually because operational tasks are ignored as they are not essential to the developer workflow.
In these companies the initial brief mentions either Terraform or Cloud Formation.
The developers have chosen the tooling, got as far as they could, and now need help.

Clients find it quite easy to reel off a large number of tasks in the operational column.
Operational tasks are the ones where you can decrease friction for everyone working in your environment.
If it takes three hours to manually test a database restore, that's an awful lot of friction for individuals in your team.

On the infrastructure side, I have visited companies who place a higher priority on IaC to recreate their production environment.
They will, of course, almost certainly never run that code to recreate production VPCs.
But because that was their focus, they choose an automation tool which favoured that type.

---
layout: post
title: Deduce You Way To DevOps Tooling
permalink: /ansible/how-to-choose-devops-tooling
tags: ansible,puppet,terraform,cto,devops,engineering,deduction,operations,imperative,declarative
---

(State the surface problem, terraform v ansible)

(Identify the real problem, no discipline to choose tools on merit)

(Why this matters)

---

**(State the surface problem, terraform v ansible)**

Are technology people inherently irrational when faced with making choices?
I have pondered this for many years, with increasing frequency as the lifetime of software tools has rapidly decreased.
There are so many discussions about bringing the Next Big Thing into the DevOps toolset.
Gone are the days when everything could be done with Bash and Perl.

But what is it that makes some of us so vehemently for and against particular software tools?
More pressing, how can we become more effective at choosing software tools?
I am witnessing too many Ansible v Terraform v Puppet arguments, that I am pushed into defining a method to guide the choice.
In this post I will describe this method using automation tooling as the topic.

**(Identify the real problem, not discipline to choose tools on merit)**

I asked some artist friends of mine to recommend a sketch pad for ink drawings.
Immediately they fired a volley of questions at me about where, when and how I'd be sketching.
What struck me was how familiar their line of questioning was, to software development.
We summarised my goal in the user story:

```
I would like to sketch the people and places I see while travelling to DevOps clients,
as I often have short amounts of free time sitting in cafes, or train stations, etc.
```

They then outlined the main requirements and constraints, distilling them down till everyone was in agreement:

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
We have our chosen tool and we try to justify its use in every case we come across.
We are often guilty of wilfully neglecting the user story currently in play.

My artist friends closed off by removing from the set of possibilities any products that didn't meet the requirements.
They presented me with this:

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

>**_Technology_** _("science of craft", from Greek τέχνη, techne, "art, skill, cunning of hand"; and -λογία, -logia[2]) is the collection of techniques, skills, methods, and processes used in the production of goods or services_


When engineering in a DevOps environment, the broader _and_ deeper your toolbox, the better placed you are to pick the right tool for the job.
So let's take this approach to automation tooling.

**(offer the solution)**

#### List Your Needs
When I start working with a client the first job is to figure out what they really need to achieve, their DevOps user story.
When thinking about automation I split tasks into two categories, infrastructure tasks and operational tasks.
Infrastructure tasks are creating and destroying things, ALBs, instances, VPCs, etc.
Operational tasks are everything else.

With the client we fill out this table with all the tasks they currently do, or would like to do.
For some we get started by listing their existing run-books.
We always include people in the conversation who are actually doing the work, they are the ones who really know what's going on.
I then tend to add in all the tasks they didn't realise they should do.

Infrastructure Tasks | Operations Tasks
--- | ---
Create VPC<br>Create subnet<br>Destroy RDS instance<br>Create/destroy ephemeral testing env | Manage ephemeral testing env's (query, extend)<br>Modify ECS clusters<br>Database restore testing<br>Flip database cluster nodes for patching<br>Update ECS task definitions<br>Rotate AWS IAM deployment api keys<br>enable sales to spin up demo env's

The key understanding here is that infrastructure tasks suit declarative tooling (Terraform, Cloud Formation, etc), and operational tasks suit imperative tooling (Puppet, Ansible, etc).

With infrastructure tasks you don't care _how_ things are done, just that the resulting state is correct.
If you want to destroy a test environment in AWS, you only care that no infrastructure is left standing.
It doesn't matter if one or other EC2 instance is destroyed first.

On the other side, operational tasks are those where you generally do care about how things are done.
Perhaps there is an operational task to test a database backup can be restored.
This would be done with an imperative tool, through an ordered series of state changes.
If you imagine a 'restore testing' run-book, you have to complete each step to get to the end state.

#### Deduce Tooling Paradigm
The final tool choice should be guided by reasoning, and we take the first choice here.
Our goal with DevOps is all about reducing friction around the tasks we do.
Based on the above tasks table we can summarise and try and reasonably-guess some quantities.
The following numbers are based on a fictitious 20-dev web company, using Agile.

 | Infrastructure Tasks | Operations Tasks
--- | --- | ---
Task Count | 4 | 7
Manual Task Hours | 6 | 8
Weekly Run Hours | 1 | 40
Monthly Run Hours | 4 | 160
Task Type Weighting | 2.4% | 95.7%
Suitable paradigm | declarative, imperative | imperative

_(imperative is suitable for infrastructure tasks simply because in a logical sense imperative programming can define any order, and therefore can build infrastructure)_

While there are many ways to slice it, any CTO or team lead will be concerned by operational hours spent on repetitive tasks.
The above weighting example (heavily operational) is, by definition, most DevOps environments.

An interesting side note here.
Many web startups call for DevOps engineers when they get to about year two or three.
Having got that far with developers being able to stand up infrastructure and CI/CD pipelines.
At this two or three year mark, a typical startup has built a revenue stream that must be protected (with app reliability, scalable performance, etc).
It is often a great challenge to a company to perceive the change in dynamic to a more operationally focused set of needs.

#### Push Me Pull Me
The next deduction is what delivery method makes most sense.
When creating things (EC2 instances, load balancer, etc) pushing change is the only way.
So declarative tooling must use the push method of delivery, and this implies it must run from a control node (laptop, instance).
Nice and easy.

Operational tasks are also fairly easy to assign a delivery method, with one exception.
For active resources (instances, etc) an agent process can pull configuration changes.
The conundrum here is that you initially _push_ this agent onto the resource, meaning you already have a push based automation tool.

For CTOs/leads the question here is about organisational structure.
My initial probing when arriving in a company using multiple automation tools is:

* Are infrastructure and configuration management distinct teams with little or no shared code?
* Is there a compelling reason to run two, potentially conflicting, automation tools?

Fewer technologies in-play makes for a leaner more efficient DevOps environment.
However, a warning sign that too few tools are being used is the number of hacks and workarounds being used to overcome unsuitable tooling.
As someone once said, "Make things as simple as possible, but no simpler."

Here we define the delivery method as a choice of reason:

 | Infrastructure Tasks | Operations Tasks
--- | --- | ---
Task Type Weighting | 2.4% | 95.7%
Suitable paradigm | declarative, imperative | imperative
Delivery Method | push | push, pull

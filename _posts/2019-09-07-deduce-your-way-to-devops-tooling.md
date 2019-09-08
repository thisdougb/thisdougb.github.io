---
layout: post
title: Deduce Your Way To DevOps Tooling
permalink: /devops/2019-09-07-deduce-your-way-to-devops-tooling
tags: ansible,puppet,terraform,chef,cto,devops,engineering,deduction,operations,imperative,declarative,automation,linux,aws
---
### Deduce Your Way To DevOps Tooling
Are technology people inherently irrational when faced with making choices?

I have pondered this for many years, with increasing frequency as the lifetime of infrastructure tooling has rapidly decreased.
We seem to be in a perpetual state of bringing the Next Big Thing into the DevOps toolset, we are never content.
Gone are the days when everything could be done with OS packaged Bash and Perl.

Within this state of flux, what is it that makes some of us so vehemently for and against particular software tools that are seemingly disposable?
More pressing, how can CTOs and platform leads become more effective at driving tooling choices?
I am witnessing too many subjective Ansible v Terraform v Puppet arguments, that I am pushed to define a method to guide this decision making process.
In this post I will describe this method using art and automation tooling.

#### Life Imitates Art
I asked some artist friends of mine to recommend a sketch pad for ink drawings.
Immediately they fired a volley of questions at me about where, when and how I'd be sketching.
What struck me was how familiar their line of questioning was, to software development.
We summarised my goal in the user story:

>I would like to sketch the people and places I see while travelling to DevOps clients,
as I often have short amounts of free time sitting in cafes, or train stations, etc.

They then outlined the main requirements and constraints, distilling them down till everyone was in agreement:

* Sketchbooks for travelling must be low-hassle and robust (hardback, no greater than A5 size).
* Type of pen should be available to buy in most medium sized towns.
* Type of pen ink must be fast drying.
* Pens must not leak ink.
* Sketchbook paper should be of sufficient weight to hold the ink.


Quite far into the conversation I realised no-one had even mentioned a brand name.
My experience of technology discussions is that from the outset, you already know what everyone is advocating (Terraform or Ansible, Ruby or Python, etc).

Technology people often seem to start with the answer, and work backwards to justify it.
This, I feel, is what DevOps tooling discussions are really about.
Advocating for our favourite tool in every case we come across, often simply to validate to ourselves we follow the right tribe.
We, myself included, are often guilty of wilfully neglecting the user story or current context.

My artist friends closed off by removing from the set of possibilities any products that didn't meet the requirements.
They presented me with this:

* Sketchbooks to consider are A and B, pens are X, Y, and Z.
* The paper in sketchbook A works best with pen Y.
* The paper in sketchbook B works bext with pen X.
* Sketchbook B and pen Z is the cheapest combination.

Only at this point did subjectivity come into it.
Only at this point did any of them offer which of A, B, X, Y, and Z they actually prefered.
Any combination in this set meets my user requirements, so I am now free to pick and reject what I like on a subjective basis.

The real lesson from the art world is that tools are just tools.
To evangelise about a pen branded X, is to miss the point completely.
Likewise, to dismiss an individual tool, be it a pen or a piece of software, on subjective grounds is not only bad practice but also limiting your potential.

>**_Technology_** _("science of craft", from Greek τέχνη, **techne**, "art, skill, cunning of hand"; and -λογία, **-logia**[2]) is the collection of techniques, skills, methods, and processes used in the production of goods or services_

When engineering in a DevOps environment, the broader _and_ deeper your toolbox, the better placed you are to pick the right tool for the job.
So let's take this approach to automation tooling.

#### Workload Reality
When I begin working with a client the first job is to figure out what they really need to achieve, their DevOps user story.
Thinking about automation I split tasks into two categories, infrastructure tasks and operational tasks.
Infrastructure tasks are creating and destroying things, ALBs, instances, VPCs, etc.
Operational tasks are everything else.

I ask the client to fill out this table with all the tasks they currently do, or would like to do.
For some we get started by listing their existing run-books.
We always involve the people who are actually doing the work, they are the ones who really know what's going on.
I then tend to add in all the tasks they didn't realise they should do.

| Infrastructure Tasks | Operations Tasks |
| --- | --- |
| Create VPC | Modify ECS clusters |
| Create subnet | Database restore testing |
| Destroy RDS instance| Manage ephemeral testing env's (query, extend)|
| | Flip database cluster nodes for patching | 
| | Update ECS task definitions |
| | Rotate AWS IAM deployment api keys | 
| | Enable sales-reps to spin up demo env's |

The key understanding here is that infrastructure tasks suit declarative tooling (Terraform, CloudFormation, etc), and operational tasks suit imperative tooling (Puppet, Ansible, etc).

> _Declarative approach_: You ask a colleague to go out and fetch your lunch, they come back with a coffee and sandwich. The coffee is lukewarm, your colleague says the coffee started going cold while they were queuing for the sandwich.

With infrastructure tasks you don't care _how_ things are done, just that the resulting state is correct.
If you want to destroy a test environment in AWS, you only care that no infrastructure is left standing.
It doesn't matter if one or other EC2 instance is destroyed first.
Great for infrastructure, but not the best for lunch orders.

> _Imperative approach_: You go out for your own lunch. You buy the sandwich first and then the coffee, because you know this ensures the coffee will still be hot when you get back to your desk. You deliberately ordered the path to the end state.

On the other side, operational tasks are those where you generally do care about how things are done.
Perhaps there is an operational task to test that a database backup can be restored.
This would be done with an imperative tool, through an ordered series of state changes.
If you imagine a 'restore testing' run-book, you have to complete each step to get to the end state.

#### Deduce The Tooling Paradigm
Tool choice should be guided by reasoning, and we take the first choice here.
Our goal with DevOps is all about reducing friction around the tasks we do.
We can summarise the task list and give a sense of the person-hours required.
The following numbers are based on a fictitious 20-dev web company, using Agile.

| | Infrastructure Tasks | Operational Tasks |
| --- | --- | --- |
| Task Count | 4 | 7 |
| Manual Task Hours | 6 | 8 |
| Weekly Run Hours | 1 | 40 |
| Monthly Run Hours | 4 | 160 |
| Task Type Weighting | 2.4% | 95.7% |
| Suitable paradigm | declarative, imperative | imperative |

_(imperative is suitable for infrastructure tasks simply because in a logical sense imperative programming can define any order, and therefore replicate declarative programming)_

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

Operational tasks are also fairly easy to assign a delivery method.
If the list of operational tasks include creating any infrastructure (eg, testing database restores), then the push method is the reasoned choice.

There is an exception to this.
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

| | Infrastructure Tasks | Operational Tasks |
| --- | --- | --- |
| Task Type Weighting | 2.4% | 95.7% |
| Suitable paradigm | declarative, imperative | imperative |
| Delivery Method | push | push, sometimes pull |

The push delivery method wins in most cases for the combo of infrastructure and operational tasks.
When agents pull changes, you must always add additional infrastructure to monitor and check the agents.

#### Decision Time
In my example of a web company maturing to a more operationally focused state, we have refined our options using  deduction:
> _Most recurring tasks will be operational, which suits the imperative paradigm, and we are required to push changes because some of those tasks also create infrastructure._

Let's list out the current crop of CCA-like tools, and cross off any not meeting requirements.
Our goal (perhaps as CTO or lead) is to arrive at a rational tooling choice, which results in a low-friction DevOps environment.

| Tool | Released By | Paradigm | Delivery | Language |
| --- | --- | --- | --- | --- |
| Ansible | RedHat | imperative | push | Python |
| Chef | Chef | declarative, imperative | push, pull | Ruby |
| ~~CloudFormation~~ | ~~AWS~~ | ~~declarative~~ | ~~push~~ | - |
| ~~Puppet~~ | ~~Puppet~~ | ~~declarative~~ | ~~pull~~ | ~~Ruby~~ |
| SaltStack | SaltStack | declarative, imperative | push, pull | Python |
| ~~Terraform~~ | ~~HashiCorp~~ | ~~declarative~~ | ~~push~~ | ~~Go~~ |

At this point we have some remaining options to choose from, all of which meet our fictional company requirements.
Refining the choice from here could be down to the skills available in the team or job market, tool maturity and community support.

It is a straightforward choice to go with Ansible in this case.
We gain tool stability and maturity, plenty of community support, and we have a ready supply of Python capable engineers.
Ansible suits operational tasks, and supports all the cloud infrastructure building we require.

Requirements for other companies may come out differently, but the process of deduction from the initial list of tasks should be the same.
The benefit of a systematic process is that it is transparent and easily documented.
It is also easy to know when you're requirements have changed.

The above table of options would look a little different if we were weighted to automating infrastructure tasks.
We'd cross off Puppet as an option, but all others would be on the table.

#### One Deductive Process, Not One Tool
DevOps tends to be an operations-heavy job, and we focus on automating _processes_.
We focus on process because this is where the largest gains of automation can be, from time savings to increasing reliability.

An objective method for choosing DevOps tooling is important.
I don't believe technology people inherently make irrational choices.
I think we're just people, and people tend to be poor at rational decision making.
We tend to go with what we know, or are biased by more recent and frequent events we're conscious of (StackOverflow search results).

Using a systematic approach to tool choice should put us all in a better place.
And if DevOps is about anything it's about reducing the manual effort at the keyboard and increasing time spent engineering simplicity.

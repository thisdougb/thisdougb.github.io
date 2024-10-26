---
layout: post
title:  "How To Fix Open Source Project Bugs"
date:   2024-10-26 11:48:44 +0000
---

Over a series of posts, I want to share the process of fixing bugs in open source infrastructure tools.
This post is the introduction to the topic.

As I write this I don't know if we found a real bug, or if it can be fixed.
But I've fixed a few open source bugs now, and trust the process described below.
So come along on the journey, if you're interested in how bugs get fixed in the tools we use every day.

## What happened one day

Unexpected Terraform errors often cause frustration, and can grow quickly to become annoying.
Because of the way Terraform works (code -> state -> real life), an unexpected error can halt your work.

A collegue asked me why a Terraform-managed AWS Aurora cluster failed to destroy, giving a confusing error.
The instances are destroyed, but when it comes to the cluster Terraform can't do it and stops.
I tested creating and destroying a cluster myself and got the same error:

```
aws_rds_cluster_instance.cluster_instances[0]: Still destroying... [id=aurora-cluster-demo-0, 9m40s elapsed]
aws_rds_cluster_instance.cluster_instances[0]: Destruction complete after 9m49s
aws_rds_cluster.default: Destroying... [id=aurora-cluster-demo]

│ Error: RDS Cluster final_snapshot_identifier is required when skip_final_snapshot is false
```

Right now, I don't even know what the `final_snapshot_identifier` is.
I also don't know what `skip_final_snapshot` is, and haven't set it to `false` in my code.

#### So what's going on, and should you care?

I hope to show you not only what's going on, but how you can fix it.
You don't need to be an experienced coder to fix bugs.
If you can clearly state your investigation and workings, an experienced coder will pair with you to create a fix.
This is the magic of open source, purpose driven cooperation.

It is in all of our interests that that the tools we use are high quality. Our companies depend on these tools.
When you run into a bug that stops your work, be sure that quite a few other people in other companies hit the same bug.
They also got stuck, frustrated, and maybe annoyed.

#### We cannot stop bugs in software.
But you can swap out frustrated for intrigued, to create an itch that you need to scratch.
Why is this error happening, is it really a bug, how complex is it, how many does it affect?

In doing this you change perspective from user to investigator.
Learning more and more as you travel towards understanding the problem. 
And then change perspective again to solver.
As a solver of things you consider the evidence in front of you, and find options to unstick everyone.
Once more change and your pragmatic perspective brings a rational appraisal of the options, the time required versus the benefit.

Finally, hopefully, a fix is created and merged.
They are cheering in the streets, your name is whispered in awe in forums at the far reaches of the Internet.

OK, maybe not that last part. But now you're part of the magical tech-industry phenomenon that is _open source_.
Your team mates will look at you in a new light.
Your boss will know you are the one who gets things done, a good bet for more interesting project work.

You travelled from user to contributer, and became an active particpant in shaping the tools we all use.
This is engineering.

## The Process

Most of the tools we use in DevOps are open source.
The majority of people who maintain them are giving their time and effort for no financial reward.
But like a lot of life, what is essential is invisible to the eye.

I am writing this at the weekend, using my own time and money (AWS costs $$).
Though my reward is greater, to me, than money, it is continued involvment in the magic of open source.
Experiencing, for a moment in time, that a handful of people from across the planet join together over a shared sense of purpose.
We simply want to be unstuck.

So enter into this process with repsect for everyone you encounter.
You don't know what kind of day, week, or year, someone else is having.
You may or may not speak the same language, so turn your tolerance dial up to eleven.
People giving their time may be unfamiliar with coding or inexperienced, help people understand and learn.

### Practical Steps

Bug fixing in open source projects is a fairly simple process.
Here's a summary of the steps that we'll expand on as we walk through fixing the above error.

It is easy to get carried away, and believe you've found a critical bug.
Try everything you can think of to find out what local mistake you've made which disproves your notion that this is a bug.
Try a different system, and different method, ask someone else to try it.
Read the docs again, to check if you're just mistaken on the behaviour of the software.

#### User: Clearly state the problem

In the plainest terms state what you are trying to do, what was expected, and what actually happened.

This is a bit like a pitch, not just for the problem but also your pitch showing your due diligence and clear communication skills.
The higher the quality of your opening issue statement, the more likely it is that experienced people will show up and give their time to your problem.
It doesn't need to be deeply technical, just a clearly stated description of the problem.

#### Investigator: Go deep and wide

Become a subject matter expert on the problem.
By probing, testing, learning, you can find configurations when the problem occurs and when it doesn't.
Width is as useful as depth.
Find the scope of the problem.
Is it all Aurora clusters or just postgres, does this affect standalone instances, etc?

#### Solver: There's more than one way to do it

Problems can often be solved in multiple ways.
Sometimes better documentation mitigates poor code, or maybe a code fix is required, maybe both.
There's a third way, for those not experienced with coding.

If you understand the system (eg, creating and deleting Aurora clusters) your solution may be a description of the logical steps that should happen.
This is valuable to pair with someone who could implement that change in code.

I always find three is the magic number, if possible.

#### Pragmatist: Between the horns of a dilemma

The _best_ way may not be the most technically sound one, for a bunch of valid reasons.
This is why you should always trying and find more than one way to solve a problem.
It forces you to look from different points of view, and make more pragmatic choices.

#### Fixer: The simplest part

If the previous stages have been worked through with enthusiasm, then the fixing stage should be the simplest.
You may or may not be the fixer, but it will depend on all of the previous stages.

The fix will follow the projects guidelines, and be reviewed by a small team of maintainers. 
This can take time.
The clarity and quality of the merge request (fix) is key here, make it easy for the reviewer.

## How it went

I'll post links to the stages here, as I do them.

---

-- started on a sunny Saturday in Edinburgh.

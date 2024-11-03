---
layout: post
title:  "Clearly State Terraform AWS Provider Bug"
date:   2024-11-03 11:48:44 +0000
---

The first part of this series is [here]({% post_url 2024-10-26-fix-open-source-project-bugs %})

In the plainest terms state what you are trying to do, what was expected, and what actually happened.

This is a bit like a pitch, not just for the problem but also your pitch showing your due diligence and clear communication skills.
The higher the quality of your opening issue statement, the more likely it is that experienced people will show up and give their time to your problem.
It doesn't need to be deeply technical, just a clearly stated description of the problem.

### Begin at the end

If we look into the future we can imagine that we have found a bug and then someone fixed it.
But if we look at [reality](https://github.com/hashicorp/terraform-provider-aws/issues) there's only a slim chance this will actually happen.

Today (_09:02 Sunday 3rd Nov 2024_) there are 3,428 open issues in the terraform-aws-proiver repository. There are 393 open pull-requests.
I don't know how many maintainers there are, but that's a huge backlog of things seeking attention.

A Github repo is not a marketplace, so you shouldn't try and game the maintainers into picking up your issue.
What is valued is honesty, accuracy, and clarity.
If these values come through in how you prepare your issue/bugfix, then you make it easier for maintainers and more likely someone will pick it up.

The Terraform AWS Provier repository has a [form](https://github.com/hashicorp/terraform-provider-aws/issues/new?assignees=&labels=bug&projects=&template=00_bug_report.yml&title=%5BBug%5D%3A+) to help you provide the most useful information.

Producing a clear and accurate bug report is the goal of this part of the process.

### What really happened

It can be tricky for technical people to state things clearly and simply. We all want to provide lots of information we think will be useful. I include myself in the group that often over-complicates explanations.

With a bit of thought, you can whittle down the words to be succint. The following four sections are from the _New Issue_ form, for the Terrafom AWS provider. These sections are a bit like newspaper headlines and summaries, someone is going to make a decision to be interested or not based on what you put here. We’re all human after all.

I often write down what happened, and then reduce the wording away from a computer in a more human environment. In a cafe, with a coffee, I try and rewrite using the simplest words and sentences that accurrately describe what happened. Remember again, not everyone’s first language is the same as yours.

The project maintainers want you to report the behaviour experienced, not opinion or solutions (at this point). They are trying to operate efficiently by standardising issue reporting.

#### Expected Behaviour

When trying to destroy a freshly created (vanilla) RDS Aurora cluster, the cluster should be destroyed.

#### Actual Behaviour

Destroying the cluster instance is successful, but destroying the cluster itself fails because no `final_snapshot_identifier` value is configured in code. I must manually destroy the cluster via the console or cli.

#### Relevant error snippet

```
aws_rds_cluster_instance.cluster_instances[0]: Still destroying... [id=aurora-cluster-demo-0, 9m40s elapsed]
aws_rds_cluster_instance.cluster_instances[0]: Destruction complete after 9m49s
aws_rds_cluster.default: Destroying... [id=aurora-cluster-demo]

│ Error: RDS Cluster final_snapshot_identifier is required when skip_final_snapshot is false
```

#### Bug title

I often decide on this last, because only after really thinking about the problem can I summarise in a few words.

> RDS Aurora cluster fails to destroy because `final_snapshot_identifier` value is required

### Are you unique?

Now we have classified our problem, it’s time to find out if a bug has already been raised. Because we have our own clear statement, we can objectively compare to any existing/past issues we find. 

I found this issue (Dec 2017) https://github.com/hashicorp/terraform-provider-aws/issues/2588 , which had a useful snippet in the comments:

>For me, I was able to add `skip_final_snapshot`, re-apply, and then I was able to destroy without issues. The issue is indeed the state is holding the default value of false, because there was not one specified.

I also found this https://github.com/hashicorp/terraform/issues/15530

And this https://github.com/hashicorp/terraform-provider-aws/issues/92

From reading through the issues, it seems that this is a long-standing problem affecting quite a few people. There are workarounds (editing statefile, manual deletions, etc), but I don’t see any proposed fix. Over eleven years, no-one has submitted a PR as far as I can see. Only the first issue remains open, the others have been closed due to inactivity.

If you read the detail of the issue comments, there are subtle variations. One thread relates to cluster _instances_ failing to destroy (not a match for us), for example.

Issue [2588](https://github.com/hashicorp/terraform-provider-aws/issues/2588) seems to be the principle bug report. Although it’s not an exact match either, because the bug-reporter is using these settings:

```
  skip_final_snapshot = true
  final_snapshot_identifier = "whatever"
```

We haven’t specified either of these, but it seems too close to be a coincidence. So, at the very least, we should refer to this issue if we decide a new bug report is required.

### What we now know

I’m reasonably sure that this is a bug, so I am deciding to continue. The similarity to reported issues, and the types of workarounds, suggest this isn’t expected behaviour.

Even if this doesn’t turn out to be a logic/code bug, the documenation may need updating to prevent so many people experiencing the problem.

What I am unsure about is whether to file a new bug report, or add to [2588](https://github.com/hashicorp/terraform-provider-aws/issues/2588). So in the next step I’ll go a bit deeper and look at the code to try and figure out why we’re seeing this behaviour.

If I do create a new issue, then I now have thought through all the details in order to submit a decent bug report.

---

-- started near Ettrickbridge, Scottish Borders.

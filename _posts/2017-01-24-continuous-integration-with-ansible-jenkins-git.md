---
layout: post
title: Continuous Integration With Ansible Jenkins Git
permalink: /ansible/2017-01-24-continuous-integration-with-ansible-jenkins-git
tags: ansible,jenkins,git,continuous,integration,pipeline,devops,engineering
---
The benefit of sketching out your integration/deployment process (the sequence of steps), is that intuitively you will discover the tasks that can be automated.   The more tasks that can be safely automated the more 'continuous' the integration of changes to production will be.

![Continuous Integration pipeline planning](/images/ci_pipeline_planning_800x600.jpg)

If you don’t yet have an integration pipeline then creating one from scratch can be confusing.   Particularly so if you don’t come from a development background, or are new to Git and deployment of code to production.  I am using 'production' as a catch-all term for 'the code-storage-place that must remain stable'.

What should your Source Code Management workflow look like?   What's the best repository structure, which depends on or informs  how the team share code?   Should the integration pipeline (Jenkins) be defined before or after the source code control (Git) workflow?   

To avoid paralysis in the face of multiple interrelated chicken and egg decisions, just apply some Agile thinking.   Iterate!

Begin with general-yet-basic components of your overall workflow, with the expectation of moulding them as is develops.   The important thing is to do something, even if it turns out to be not quite what you end up with.   It’s all experience and learning, which feeds back into the design as an interative process.   With open tools such as Git, Jenkins, etc, there is no definitely correct way to do things.

I'm going to describe a simple overall workflow, which will encompass making code commits and integrating them into our production repository.   This gives us a starting point and model to morph, tweak, adjust, etc.   From this we can create a tailored set of 'the rules', which contributors agree to follow to make the whole thing work.

This is not definitive in any way, nor is it intended to be 'best practice'.   It is intended merely to expose some of the human aspects of continuous integration, and raise some points for consideration.

Summary of desired workflow:

1. The Communote team want to turn on mysql query logging on their backend servers, so create a Jira ticket for the task.
2. Sam (DevOps) modifies the existing Ansible cnote-mysql-config role.
3. Sam commits this change to the shared remote repository (Git) of Communote infrastructure code.
4. The automation pipeline (Jenkins) is woken up by a git commit hook, tests the new code and, if successful, merges the new commit into the producion repository.
5. The Jira ticket for the change, containing a full audit trail, is set to completed.
6. At some later point, Chris (DevOps) runs the playbook (Ansible Tower) to execute the change on the live servers.


## What does the Git workflow look like?

To work well a Git workflow depends on people doing what they say they will do.   Developers should be lazy (aka efficient), striving for the minimal effort that provides greatest gains.   A workflow that is light and efficient, with repetitive boring tasks automated, has a good chance of success.   Stand-ups become a vital hub, because there is a real shared interest in communicating code/feature/integration state.

A good Git workflow matters, because it saves time and team conflict.   Let's start with the [Private Small Team](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project) workflow, which comes with some basic rules to make team-life easier.   Your own team rules should fit on a single A4 sheet, and pinned up with clear line of sight.   The team own the workflow, it's their responsibility maintain its effectiveness.

The Private Small Team workflow simply involves a central repository which developers pull the latest code from, and then push back changes using topic branches.   Pushing topic branches allows an automated integration pipeline (Jenkins) to test and validate changes, without potentially messing up ‘master’.

If automated testing/auditing in the Jenkins pipeline is comprehensive enough, continuous integration of topic commits into ‘master’ may be possible.   Otherwise if a manual authorisation step is required (often for compliance purposes), we can implement this as part of the workflow with a Jira ticket state change (triggering the git merge to master).


## What does a DevOps workflow look like?

Given the above workflow, what would a DevOps change look like on the command line?   Usually we’d have something like a Jira ticket to track the work being done, and the scope of the work would be specific and clear (visualise your commit message).

We setup our dev environment by refreshing or cloning the master repo, this is a copy of the latest 'production' code base.
```
$ git clone gitolite@gitserver:communote
$ cd communote
```

Branch off with a suitable name (naming is part of 'the rules').   The branch name is prefixed with the ticket ID, which we can use later in the integration pipeline to match the commit to a valid Jira ticket.
```
$ git checkout -b jra-780-mysql-general-log
```

Make, test and commit the changes locally.   The commit message format is also part of 'the rules', and poor commit messages should be rejected later on in the pipeline.
```
$ vi roles/mysql-server/tasks/main.yml
$ git add roles/mysql-server/tasks/main.yml
$ git commit -m ‘JRA-780 Enable general query log in mysql server, to log all sql statements to /var/lib/mysql/general.log’
```
Push the changes back to the centralised repo, which creates the topic branch on origin.
```
$ git push -u origin jra-780-mysql-general-log
<...snipped...>
To gitolite@gitserver:communote
 * [new branch]      jra-780-mysql-general-log -> jra-780-mysql-general-log
Branch jra-780-mysql-general-log set up to track remote branch jra-780-mysql-general-log from origin.
```
The state at this point is that the central 'master' repo branch remains untouched (therefore stable).  The central repo (origin) now has a new branch called jra-780-mysql-general-log, which will now undergo a series of manipulations and tests by our Jenkins pipeline.

## The Integration Pipeline
Jenkins, like Git, starts off pretty much as a blank canvas.   There is no correct pipeline, only combinations of tasks that are suitable for your desired workflow idea.   In addition, different products and even different types of changes may result in a handful of Jenkins pipelines.   Planning is important, as is refactoring when things are no longer obvious or have too many logical branches.

For our purposes we want to keep Jira updated with progress (audit trail), and test our Ansible playbook components.   If any step fails then we simply stop and notify the Jira ticket, which notifies the relevant people.   On an integration failure, the code is fixed and re-pushed to the topic branch.

The integration itself (merging the change into production) is logically tied to the Jira ticket status.  If all tests pass, then the status will be OK and the merge can happen automatically.   So long as the Jira workflow is setup such that no-one else can modify the workflow status, then it shouldn't be possible to circumvent our automated testing.

Jenkins pipelines are built as code, stored in a repo, and pulled automatically when the job is run.   The pseudo-code of our Jenkins pipeline:
```
#!groovy

node { 
	stage('Begin') {
	    // match commit to Jira ticket
		// slackSend message: "${JIRA_TICKET}: begin ${pipeline} with ${featurebranch}"
	}
	stage('Files') {
		// for each yml file:
		//     ansible/yaml lint file
		//     detect disallowed modules
		//     detect cleartext passwords
		//     ...
		//     add file role to roles_list
		// update Jira
	}
	stage('Roles') {
		// for each role:
		//     unit test role on vm
		//     add role playbook test to playbook_list
		// update Jira
	}
	stage('Playbooks') {
		// for each test_playbook:
		//     run test_playbook on vm
		// update Jira
	}
	stage('Git Merge Check') {
		// git merge feature branch to master
		// update Jira
	}
	stage('Final') {
		// Send PR to merge to master
		// update Jira
		// slackSend message: "${JIRA_TICKET}: ${pipeline} ended successfully with ${featurebranch}"
	}
}
```

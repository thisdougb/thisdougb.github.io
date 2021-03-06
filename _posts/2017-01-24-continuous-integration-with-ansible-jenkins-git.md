---
layout: post
title: Continuous Integration With Ansible Jenkins Git
permalink: /ansible/2017-01-24-continuous-integration-with-ansible-jenkins-git
tags: ansible,jenkins,git,continuous,integration,pipeline,devops,engineering
---
The benefit of sketching out your integration/deployment process (the sequence of steps), is that you are forced to start with the general and then define the detail.   It is more intuitive to define the sequence of steps as a series of tasks (code, commands) and transitions (of state, authority, completeness, etc).   The more tasks and transitions that can be safely automated the more 'continuous' the integration of changes to production will be.   

I am using 'production' as a catch-all term for *the code-storage-place that must remain stable*.   In addition, consider that some code changes may suit automated integration, while others don't.

![Continuous Integration pipeline planning](/images/ci_pipeline_planning_800x600.jpg)
*Continuous Integration pipeline planning*

This post is not intended as a definitive guide to continuous integration workflows.   It is intended merely to bring some familiarity to the whole business of automated code release, and of where Git/Jenkins/Jira fit in.   In particular the release of Ansible code, as many people come to DevOps and automation with a steep 'dev' learning curve.

* What should your Source Code Management workflow look like?   
* What's the best repository structure, which depends on or informs how the team share code?   
* Should the integration pipeline (Jenkins) be defined before or after the source code control (Git) workflow?

To avoid paralysis in the face of multiple interrelated chicken and egg decisions, just apply some Agile thinking.   *Iterate!*

To work well, a CI workflow depends on people doing what they say they will do.   DevOps people should be lazy (aka efficient), striving for the minimal effort that provides greatest gains.   A workflow that is light and efficient, with repetitive boring tasks automated, has a good chance of success.

Stand-ups become a vital hub, because there is a genuine need to communicate and coordinate code/feature/integration state.   CI pipelines are simply tools to help humans be more effective.   People are an essential element to continuous integration, whether actioning transition states, iterating workflow improvements, or communicating effectively.

Begin with the basic components of your overall workflow (*code -> git repos -> testing -> prod*), with the expectation of moulding them as ideas develop.   It’s all experience and learning, which feeds back into the workflow design as an iterative process.   With open tools such as Git, Jenkins, etc, there is no definitely correct way to do things.

Summary of desired workflow:

1. *The Communote team want to turn on mysql query logging on their backend servers, so create a Jira ticket for the task.*
2. *Sam (DevOps) modifies the existing Ansible cnote-mysql-config role.*
3. *Sam commits then pushes this change to the shared remote repository (Git) of Communote infrastructure code.*
4. *The automation pipeline (Jenkins), woken up by a git hook, tests the new code and, if successful, merges the new commit into the producion repository.*
5. *The Jira ticket for the change, containing a full audit trail, is set to completed.*
6. *At some later point, Chris (DevOps) runs the playbook (Ansible Tower) to execute the change on the live servers.*

## What does our Git workflow look like?

Let's start with the [Private Small Team](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project) workflow, which comes with some basic rules to make team-life easier.   Your own 'team rules' should fit on a single A4 sheet, and pinned up with clear line of sight.   The team own the workflow, it's their responsibility maintain its effectiveness in the face of corporate compliance.

The Private Small Team workflow simply involves a central repository which developers pull the latest code from, and then push back changes using feature branches.   Pushing feature branches allows an automated integration pipeline (Jenkins) to test and validate changes, without potentially messing up ‘master’.

```
  master:  --o--------------o--------------------o--------->
             |             /|\                  /
(branches)   |            / | \----------o-----o 
             |           /  |         (JRA-781) 
             |----------o   | 
               (JRA-780)    |
                            \-----o------> 
                                (JRA-783)    
                                                   o = commit
```
*Feature branches from master, then merge back*

If automated testing in the Jenkins pipeline satisfies corporate compliance, continuous integration of feature branch commits into ‘master’ may be possible.   Otherwise if a manual authorisation step is required, we can implement this as part of the workflow with a Jira ticket state change (triggering the git merge to master).

Remember, the goal is maintaining production stability rather than blindy achieving fully continuous integration.


## What would a DevOps code change look like?

Given the above git workflow, what would a DevOps change look like on the command line?   Usually we’d have something like a Jira ticket to track the work being done, and the scope of the work would be specific and clear (visualise your commit message).

We setup our dev environment by refreshing or cloning the master repo, this is a copy of the latest 'production' code base.

```
$ git clone gitolite@gitserver:communote
$ cd communote
```

Branch off with a suitable name (naming is part of 'the rules').   The branch name is prefixed with the ticket ID, which we can use later in the integration pipeline to match the commit to a valid Jira ticket.

```
$ git checkout -b jra-780-mysql-general-log
```

Make, test and commit the changes locally.   The commit message format is also part of the rules, and poor commit messages should be rejected later on in the pipeline.

```
$ vi roles/mysql-server/tasks/main.yml
$ git add roles/mysql-server/tasks/main.yml
$ git commit -m ‘jra-780 Enable general query log in mysql server, to log all sql statements to /var/lib/mysql/general.log’
```

Push the changes back to the centralised repo, which creates the feature branch on origin.

```
$ git push -u origin jra-780-mysql-general-log
<...snipped...>
To gitolite@gitserver:communote
 * [new branch]      jra-780-mysql-general-log -> jra-780-mysql-general-log
Branch jra-780-mysql-general-log set up to track remote branch jra-780-mysql-general-log from origin.
```

The state at this point is that the central 'master' repo branch remains untouched, therefore stable.  The central repo now has a new branch called *jra-780-mysql-general-log*, which will now undergo a series of manipulations and tests as coded in our Jenkins pipeline.

## The Jenkins Bit
Jenkins, like Git, starts off pretty much as a blank canvas.   There is no correct pipeline, only combinations of tasks that are suitable for your desired workflow idea.   Planning is important, as is refactoring when things are no longer obvious or have too many logical branches.

For our purposes we want to keep Jira updated with progress (audit trail), and test our Ansible playbook components.   If any step fails then we simply stop and notify the Jira ticket, which notifies the relevant people.   On an integration failure, the code is fixed and re-pushed to the feature branch.

The integration itself (merging the change into production) is logically tied to the Jira ticket status.  If all tests pass, then the status will be OK and the merge can happen automatically.   So long as the Jira workflow is setup such that no-one else can modify the workflow status, then it shouldn't be possible to circumvent our automated testing and slip something into production.

```
  master:  --o------------------o-------------------------->
             |                 /
(branches)   |                /
             |         --------------------
             |         | Jenkins Pipeline |
             |         --------------------
             |            /
             |           /
             |----------o
               (JRA-780)
                       
                      
                                                   o = commit
                                                   / = push
                                                   | = pull
```
*Zooming in, where Jenkins fits*

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
*Jenkins pipeline pseduo-code*

The final stage of our pipeline can automatically merge our tested feature branch back into the master branch, which integrates it into production.   Our master branch contains the commit history of the feature branch, so our code has a full audit trail.

```
$ git checkout jra-780-mysql-general-log
$ git pull
$ git checkout master
$ git pull
$ git merge --no-ff jra-780-mysql-general-log
$ git log --name-status
commit 04aa0899f65a39762c538c157745f09487c1493e
Merge: 5781df4 570be18
Author: Chris Communote <chris@example.com>
Date:   Wed Jan 25 11:23:27 2017 +0000

    Merge branch 'jra-780-mysql-general-log'
    
    enable mysql general query log.

commit 570be18887419b93be307f72e09115d45b72d5a8
Author: Chris Communote <chris@example.com>
Date:   Wed Jan 25 11:22:11 2017 +0000

    jra-780 Enable general query log in mysql server, to log all sql statements to /var/lib/mysql/general.log

M       roles/mysql/tasks/main.yml

```
*Manually merging the feature branch into master*

The possibilities in a Jenkins pipeline are vast.   We should design the pipeline logic, like any code, so that it's clear what's going on.   There's nothing magic about it, it's really just automating what you would have (should have!) done manually.   The tasks and sub-tasks can be realised with plugins and scripts, which drop you into a shell with access to a git checkout of the feature branch.

A continuous integration pipeline is a trade off between extra effort with the details (commit messages, code formatting, etc) and no longer having to do the boring stuff (manual integration testing, audit logging, etc).   At it's heart, though, are the people contributing code, and their feeling that the trade off is in their favour.

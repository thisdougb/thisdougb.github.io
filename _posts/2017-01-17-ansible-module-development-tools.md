---
layout: post
title: Ansible Module Development Tools
permalink: /ansible/ansible-module-development-tools
tags: ansible,devops,devsecops,engineering,development,developer,playbook,test,testing
---
Setting up your environment for Ansible module development is relatively straightforward, but there are some things which will make it easier to develop higher quality modules.

## Cowboy Style
Pulling your spurs on and using this development process is where most of us start.   The least amount of effort that returns a feeling of progress.   And I don’t think there’s much wrong with this method if you’re a complete beginner, or knocking up the briefest proof of concept.

```
$ mkdir sec_modules
$ cd sec_modules
$ mkdir library
$ vi library/sec_suid.py
$ vi test.yml
$ ansible-playbook test.yml
```

## The Better Style
Working within teams brings some additional overhead, but that overhead brings some benefits as dev projects get larger and more involved.   Well formatted code, good documentation, clear code logic, etc, all the things we know we should do but sometimes don’t.   There are many, many styles of team development, but pretty much all of it comes down to making clear and concise communications.

Clear logic in code is something which perhaps isn’t talked enough about in DevOps, so I’ll expand a little.   Complicated logic is hard to follow, hard to fault-find.   We’ve all listened to someone telling a joke where they jump backwards and forwards in time and context.   You hear all the words, but linking the detail is difficult.  It ain't funny.

Poor code logic invites similar problems.   When you return to poor quality code you wrote it can often make no sense whatsoever.   There are lots of ways to improve your code logic, but having a plan and recognising when to refactor are key.   SysAdmins moving into DevOps really need to assimilate this concept, for their own and others sanity.

![Developing Ansible modules starts with a plan](/images/ansible_module_dev_coffee.jpg)
*Developing Ansible modules starts with a plan, and a coffee.*

Ansible provides a couple of tools to help with module development.   To set these up, clone the Ansible repo somewhere outside of your dev area.   This will give access to module testing tools, but keeping that repo away from the project .git instance.
```
$ git clone git@github.com:ansible/ansible.git ~/
$ echo alias testmodule="~/ansible/hacking/test-module" >> ~/.bashrc
$ echo alias validatemodule="~/ansible/test/sanity/validate-modules/validate-modules" >> ~/.bashrc
$ source ~/.bashrc
```
The test-module tool lets you run modules without the whole Ansible baggage.   It's quicker and simpler to run modules this way, when developing.
```
$ testmodule -m library/sec_suid.py -a 'exclude_paths=/opt/communote'
***********************************
RAW OUTPUT

{"msg": "exclude paths /opt/communote: ", "invocation": {"module_args": {"exclude_paths": "/opt/communote"}}, "changed": false, "change": true}


***********************************
PARSED OUTPUT
{
    "change": true, 
    "changed": false, 
    "invocation": {
        "module_args": {
            "exclude_paths": "/opt/communote"
        }
    }, 
    "msg": "Will exclude paths /opt/communote: "
}
```
The validate-modules tool is effectively a linter for Ansible modules.   It checks the structure of the module, but it's still encumbent on the developer to write useful/great/appropriate content.
```
$ validatemodule library/sec_suid.py 
============================================================================
library/sec_suid.py
============================================================================
ERROR: No EXAMPLES provided
ERROR: No RETURN documentation provided
ERROR: GPLv3 license header not found
```
Too much process and documentation is often as bad as too little.  These simple things can be used to build good behaviour when developing Ansible modules.  They can be built into testing scripts and incorporated into delivery pipelines.

Life and deadlines are way too short to put up with low quality code.

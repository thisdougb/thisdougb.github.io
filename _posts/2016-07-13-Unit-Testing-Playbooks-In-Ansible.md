---
layout: post
title: Unit Testing Playbooks In Ansible
permalink: /ansible/unit-testing-roles-continuous-integration
tags: ansible,role,roles,playbook,play,devops,engineering,continuous,integration,jenkins
---
I confess that I was not excited about the idea of unit testing Ansible roles.
Roles written in the minimalist sense should fail early using the Ansible framework.
A service is either present or not, and Ansible will tell you.
To add extra layers of testing seems to introduce complexity and duplication, which may itself fail.

But while talking to a client about their Ansible journey, I've become more receptive to unit testing of roles.
With many DevOps engineers co-maintaining a bunch of roles the company found errors were creeping in. 
Enough errors that similar roles were being written from scratch for each project, deemed more expedient than unpicking role-spaghetti.
The technical-debt danger applies to DevOps just as much as to app development.

I blogged about the dangers of [bloated roles](https://thisdougb.github.io/ansible/avoiding-bloated-roles "Bloated roles in Ansible"), and that's certainly one route to playbook unhappiness.
But my clients situation caused me to ponder if there really was a case for unit testing after all.

Imagine the following minimal role called *baseHttpdPackage*, committed by DevOps engineer @thisdougb.

```
---
# file: provisioning/roles/baseHttpdPackage/tasks/main.yml

- name: Ensure Apache web server is installed
  yum:
    name: httpd
    state: present

- name: Ensure Apache web server is running
  service:
    enabled: yes
    name: httpd
    state: started
```

Great role, does a simple job and no more.
It will fail early if httpd is not available, or not started.
The role is reusable anywhere, by anyone, and we will hardly ever need to alter it.

But another DevOps engineer working on a different project subsequently commits a change to our shared repo of roles.
Let's call this other engineer @thisdevilb.
Here's what @thisdevilb commits:

```
---
# file: provisioning/roles/baseHttpdPackage/tasks/main.yml

- name: Ensure Apache web server is installed
  yum:
    name: httpd
    state: present

- name: Ensure Apache web server is running
  service:
    enabled: yes
    name: sshd
    state: started
```

See the contrived clanger?
The mistake on the second last line won't cause the play to fail, but it's clearly not our intended state.
In a busy DevOps team co-authoring large plays, this sort of mistake happens.
If this rolls out to our production environment, or impacts our dev environment, it's not good.
So this appears to be a very good case for running unit tests on roles.

As always with unit testing it should be a fairly easy and well defined process.
Tests should be read only, never making changes to state.
They should test only a single associated task, so we shouldn't get carried away with complexity.
And they should be tagged so we can run them in isolation.

I prefer the tight association of unit test and task in the same .yml file, *'task > unitTest > task > unitTest'*.
This suits my preferred style of building roles, thought of as components or discrete units of work.
Despite effectively doubling the size of the role with unitTest code it preserves readability.

```
---
# file: provisioning/roles/baseHttpdPackage/tasks/main.yml

- name: Ensure Apache web server is installed
  yum: 
    name: httpd 
    state: present

- name: unitTest - Apache web server is installed
  command: /bin/systemctl status httpd
  register: result
  change_when: false
  failed_when: "'Loaded: not-found' in result.stdout"
  tags:
    - unitTests

- name: Ensure Apache web server is running
  service: 
    enabled: yes
    name: httpd 
    state: started 

- name: unitTest - Apache web server is running
  command: /bin/systemctl status httpd
  register: result
  changed_when: false
  failed_when: "'Active: active (running)' not in result.stdout"
  tags:
    - unitTests
```

Tags are important here, because we can use them in our continuous integration framework.
On a commit we trigger Jenkins to run the entire playbook on a virtual instance.
This checks .yml syntax, and that all tasks in the playbook run.

A second run of just the unit tests, *'--tags unitTests'*, checks we really do have the configuration we intended.
This is particularly useful for automated integration testing.

```
$ ansible-playbook provisioning/site.yml --tags "unitTests"

PLAY [webserver] ***************************************************************

TASK [setup] *******************************************************************
ok: [web1]

TASK [webNodePackages : unitTest - httpd installed] ****************************
ok: [web1]

TASK [webNodePackages : unitTest - httpd is running] ***************************
fatal: [web1]: FAILED! => {"changed": true, "cmd": ["/bin/systemctl", "status", "httpd"], "delta": "0:00:00.009078", "end": "2016-07-14 05:44:48.836257", "failed": true, "failed_when_result": true, "rc": 3, "start": "2016-07-14 05:44:48.827179", "stderr": "", "stdout": "● httpd.service - The Apache HTTP Server\n   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)\n   Active: inactive (dead) since Wed 2016-07-13 14:57:07 EDT; 14h ago\n", "warnings": []}

PLAY RECAP *********************************************************************
web1                       : ok=2    changed=1    unreachable=0    failed=1   
```

There are many ways to automate testing of playbooks, and include them in your continuous integration pipeline.
The most important thing is to take an approach which fits the team working style, so the barriers to comprehensive unit testing are low.
And if you're beginning with unit testing then your developers will undoubtedly have some sage advice to pass on.

---
layout: post
title: Unit Testing Playbooks In Ansible
permalink: /ansible/unit-testing-roles-continuous-integration
tags: ansible,role,roles,playbook,play,devops,engineering,continuous,integration,jenkins
---
I confess that I am not an advocate of testing Ansible roles.
Roles written in the minimalist sense should fail early, it's the Ansible way.
To add extra layers of testing seems to simply introduce complexity and duplication, which may itself fail.

But while talking to a client about their Ansible journey, I've become more receptive to unit testing of roles.
With many DevOps engineers working on a bunch of roles the company found errors were becoming noticable enough that roles were duplicated from scratch.
I blogged about the dangers of [bloated roles](https://thisdougb.github.io/ansible/avoiding-bloated-roles "Bloated roles in Ansible"), and it's certainly a major cause of playbooks not playing ball.
But my clients situation caused me to ponder if there really was a case for unit testing after all.

Imagine the following minimal role called *httpdPackage*, committed by DevOps engineer @thisdougb.

```
---
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
The role is reusable anywhere, by anyone.

But another DevOps engineer working on a different project subsequently commits a change.
Let's call this other engineer @thisdevilb.
Here's what @thisdevilb commits:

```
---
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

See the clanger?
The mistake on the second last line won't cause the play to fail, but it's clearly not our intended state.
And to be fair, in a complex setup with included files and variables, and multiple engineers, this sort of mistake happens.
If this rolls out to our production environment, or even taking out our dev environment, it's not good.


So this appears to be a very good reason for running unit tests on roles, particularly in many-hands environments.
There should be some consistency when writing unit tests.
They should be read only, never making changes.
They should also test only a single associated task, so we shouldn't get carried away with complexity.
And they should be tagged so we can run them in isolation.

```
---
- name: Ensure Apache web server is installed
  yum: 
    name: httpd 
    state: present

- name: unitTest - httpd installed
  command: /bin/systemctl status httpd
  register: result
  failed_when: "'Loaded: not-found' in result.stdout"
  tags:
    - unitTests

- name: Ensure Apache web server running
  service: 
    enabled: yes
    name: httpd 
    state: started 

- name: unitTest - httpd is running
  command: /bin/systemctl status httpd
  register: result
  failed_when: "'Active: active (running)' not in result.stdout"
  tags:
    - unitTests
```

Tags are important here.
We can use them to make continuous integration much more effective.
On a commit we trigger Jenkins (for example) to run a play on a virtual instance, and then run again with *--tags unitTests* to confirm we have a valid configuration.

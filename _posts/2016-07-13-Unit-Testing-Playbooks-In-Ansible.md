---
layout: post
title: Unit Testing Playbooks In Ansible
permalink: /ansible/unit-testing-playbooks
tags: ansible,role,roles,playbook,play,devops,engineering,continuous,integration,jenkins
---
I confess that I am not an advocate of testing Ansible playbooks or roles.
Roles written in the minimalist sense should fail early, in the Ansible way.
To add extra layers of testing seemed to simply introduce complexity, which may itself fail.

While talking to a client about their Ansible journey I think I've become more receptive to unit testing of roles.
With a large number of DevOps engineers all churning out roles they were finding conflicts were becoming common.
I blogged about the dangers of [bloated roles]{https://thisdougb.github.io/ansible/avoiding-bloated-roles "Bloated roles in Ansible"), and it's certainly a major cause of playbooks not playing ball.
But their situation caused me to ponder if there really was a situation where unit testing was a benefit rather than simply an overhead.

Imagine the following minimal role called *httpdPackage*, committed yesterday by DevOps engineer thisdougb.
```
---
- name: Ensure Apache web server is installed
  yum:
    name: httpd
    state: present

- name: Ensure Apache web server running
  service:
    enabled: yes
    name: httpd
    state: started
```
Great role, does a simple job and no more.
It will fail early if httpd is not available, or not started.

So while I am out for lunch another DevOps engineer on a different project commits a change to our roles repo.
Let's call this other engineer thisdevilb.
Here's what thisdevilb commits:
```
---
- name: Ensure Apache web server is installed
  yum:
    name: httpd
    state: present

- name: Ensure Apache web server running
  service:
    enabled: yes
    name: sshd
    state: started
```
The mistake on the second last line won't cause the play to fail.
And to be fair, in a more complex setup with included files and variables, and multiple engineers, this sort of mistake is quite possible.
If this rolls out to our production environment, or even taking out our dev environment, it's not good.



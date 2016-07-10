---
layout: post
title: Avoiding Bloated Roles In Ansible
tags: ansible,role,guide,devops
---
While working on a recent Ansible project I think I realised why roles are both fantastic and deadly.
It's not the syntax, it's the semantics.
And the problem comes from an evolution in the meaning and intent of Ansible *roles*.

When I first started using roles, the documentation gave examples of roles with names such as *webserver* and *appserver*.
Which made complete sense, in the same was as a developer role, or project manager role.
However, using this high level meaning of role to define (or hide) complex configurations is where I find things get messy.

As I built roles for real production services the list of tasks grew, and it became clear something was wrong.
It looks all neat and clean in the site.yml, but the mess is simply lurking in the ~/tasks/main.yml file.
I'm using roles as Ansible wants me to (and Ansible clearly wants us to), but the results can be ugly, sometimes very ugly.

This manifests as broken playbooks, lack of code-reuse, inflexibility, and continual tweaking.
Precisely the problems Ansible is designed to solve.
Ironic, huh?

My *webserver* role needed nginx, nginx config, firewall rules, intrusion detection scripts, etc.
But my *apiserver* role also needs the same tasks as my webserver role, except for the nginx.conf template.
I have almost 90% duplication in my role tasks/main.yml files.

```
---
# file: site.yml
- name: webserver
  hosts: webservers
  roles:
    - common
    - webserver

- name: apiserver
  hosts: apiservers
  roles:
    - common
    - apiserver
```

In the above we could have a single role with some logic around nginx.conf, but doing so loses readibility and intuitiveness.
We also break what a *role* naturally means to us; this AWS instance is a front-end webserver, I don't want the role to *sometimes* be an api server.
I think this is the crux of many messy Ansible environments in large sites, you're trying to do what Ansible tells you but it turns out less than good.

But....what if we change the word role for *component*?

Look at your hand.
Imagine the palm of your hand is the core component, and the fingers are the tasks that combine to make it a *hand*.
If my *component* is nginx, then I can think of the fingers as the stock nginx.conf, installed state and service state.
Such a component is reusable, anywhere.
My hand is always a hand, never sometimes a foot.

```
---
# file: site.yml
- name: webserver
  hosts: webservers
  components:
    - common
    - nginx
    - nginxWebConfig
    - firewall
    - firewallWebConfig
    - webIntrusionDetectionConfig

- name: apiserver
  hosts: apiservers
  components:
    - common
    - nginx
    - nginxAPIConfig
    - firewall
    - firewallAPIConfig
    - apiIntrusionDetectionConfig
```
To my mind this site.yml is much more informative, and I'm able to reuse components (roles) much more easily.
In turn, if I need to change the api server firewall rules then I won't affect the webservers group at all.
And yet if I need to change the base firewall package in my platform I only need to make that change in a single role.
Precisely what Ansible is designed to do.

So Ansible's strength is also a potential weakness.
It often appears easier just to create a fresh task list because reusing a role involves burying logic and inviting Git commit conflicts.
But when you combine Ansible's strength with its power of reusable elements (components, ie roles) you can leverage some serious efficiency gains.
Thinking in terms of components (roles) encourages reuse, saving time and avoiding coding conflicts.

If you're finding yourself continually writing roles from scratch, perhaps you're using Ansible's strength but missing out on its power.
When you think of roles as simple components, they become reusable by design.
There are strong similarities with the core object oriented programming paradigms.
I don't think Ansible Inc were wrong to call them *roles*, I simply think they've evolved to have a slightly different English language meaning in the more complex environments we work on.

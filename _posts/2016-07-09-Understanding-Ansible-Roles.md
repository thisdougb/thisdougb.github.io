---
layout: post
title: Understanding Ansible Roles
tags: ansible,role,guide,devops
---

While working on a recent Ansible project I think I realised why Ansible roles are both fantastic and deadly.
It's not the syntax, it's the semantics.
And the problem comes from a subtle change in the meaning and intent of *roles*, which appears to cause role-bloat. 

When I first started using roles, the documentation gave examples of roles with names such as *webserver* and *appserver*.
This is fine when you're learning and trying out some examples with Vagrant VMs.
However the role of a production webserver is a lot more complex, and it's here roles (precisely where we need them) get messy.

As I built roles the list of tasks grew, and it became clear something was wrong.
I'm using roles as Ansible wants me to (and Ansible clearly wants you to), but the results are ugly, sometimes very ugly.

It looks all neat and clean in the site.yml, but the mess is pushed into the ~/tasks/main.yml file.
My *webserver* role needed nginx, nginx config, firewall rules, intrusion detection scripts, etc.
But my *apiserver* role also runs an nginx proxy service.
It has almost the same list of tasks as my webserver role, except for the nginx.conf template.

```
# file: site.yml
---
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

Now, in the above we workaround this and reuse the webserver role with some logic around nginx.conf, but doing so feels ugly and loses the api role boundary.
My webserver role doesn't feel reusable for my api servers, I'm just hacking it as an exception.
I think this is the crux of many messy Ansible environments in large sites, you're trying to do what Ansible tells you but it turns out less than good.

I also think Ansible Inc started out with the right intentions, but as Ansible has become successful it's outgrown the original meaning of *role*.
What if we change the word role for *component*?

Look at your hand.
The palm of your hand is the core component, and the fingers are the tasks that combine to make it a *hand*.
If my *component* is nginx, then I can think of the fingers as the stock nginx.conf, installed state and service state.
Such a component is reusable, anywhere.
I follow this with another component (role) called nginxWebConfig, which is essentially just a template based on gathered host facts.

```
# file: site.yml
---
- name: webserver
  hosts: webservers
  roles:
    - common
    - nginx
    - nginxWebConfig

- name: apiserver
  hosts: apiservers
  roles:
    - common
    - nginx
    - nginxAPIConfig
```

My API server is easy now and I can reuse the nginx component, I only need to create a new nginxAPIConfig component.
So Ansible's strength is also a potential weakness, it often appears easier just to create a fresh task list because reusing a role involves burying logic.
But when you combine Ansible's strength with its power of reusable elements (components, ie roles) you can leverage some serious mid to long term gains.
Creating components (roles) as descrete components lets you reuse them, saving time and avoiding coding conflicts.

If you're finding yourself in an environment where you're continually writing roles from scratch, perhaps you're missing out on Ansible's power.
When you think of roles as simple components, they become reusable by design.
I don't think Ansible Inc were wrong to call them *roles*, I simply think they've evolved to have a slightly different English language meaning in the more complex environments we work on.

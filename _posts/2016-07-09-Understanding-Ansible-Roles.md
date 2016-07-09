---
layout: post
title: Understanding Ansible Roles
tags: ansible,role,guide,devops
---

While working on a recent Ansible project it became clear to me why so many people seem to struggle with roles in Ansible.
It's not the syntax, it's the semantics.
And the problem comes from a subtle change in the meaning and intent of roles. 
I think you can deduce this from the various role names used by Ansible Inc in the documentation over the years.

When I first started using roles, the documentation gave examples such as *webserver* and *appserver*.
This is fine when you're learning and trying out some examples with Vagrant VMs.
However, as the role of a webserver becomes a little more involved (a production server rarely requires only nginx) then roles and dependencies get messy.

As the list of tasks in my roles grew it became clear something was wrong.
I'm using roles as Ansible wants me to (and Ansible clearly wants you to), but the results are ugly, sometimes very ugly.
My *webserver* role needed nginx, nginx config, firewall rules, intrusion detection scripts, etc.
In the English language sense *this was the configuration required by the webserver role*.

My *API-front-end-server* role also runs an nginx proxy service.
It has almost the same list of tasks as my webserver role, but for the nginx.conf template.
Now it's possible to workaround this and reuse the webserver role with some logic around nginx.conf, but doing so feels ugly.
None of my webserver role feels reusable in for my api servers, certainly not in an easy and visibly understandable way.
I think this is the crux of many messy Ansible environments in large sites, you're trying to do what Ansible tells you but it turns out less than good.

I also think Ansible Inc started out with the right intentions, but as Ansible has become successful it's outgrown the original meaning of *role*.
What if we change the word role for *component*?
Look at your hand.
The palm of your hand is the core component, and the fingers are the tasks that combine to make it a *hand*.
If my *component* is nginx, then I can think of the fingers as the stock nginx.conf, installed state and service state.
Such a component is reusable, anywhere.
I follow this with another component (role) called webserver-nginx.conf, which is essentially just a template based on gathered host facts.

My API server is easy now, I reuse the nginx component, and only need to create a new apiserver-nginx.conf component.
Ansible's strength is also a potential weakness, it's often easy just to create a long task list and get some short term gain.
But when you combine Ansible's strength with its power of reusable elements (components, ie roles) you can leverage some serious mid to long term gains.

If you're finding yourself in an environment where you're continually writing roles from scratch, perhaps you're missing out on Ansible's power.
When you think of roles as simple components, they become reusable by design.
I don't think Ansible Inc were wrong to call them *roles*, I simply think they've evolved to have a slightly different English language meaning in the more complex environments we work on.

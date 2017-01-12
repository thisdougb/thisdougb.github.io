---
layout: post
title: Rotate Passwords with Ansible and HashiVault
permalink: /ansible/rotate-passwords-ansible-hashivault
tags: ansible,hashicorp,vault,devops,devsecops,engineering
---
__Rotating application layer passwords is hard.__   Not because changing a password in some database is difficult, it’s often only a single command.   The hard part is doing a hundred password rotations, perhaps on a daily basis, in tiered applications, with change control, compliance, auditing, etc, etc.

The benefits of automation in a large infrastructure are easy to talk about, easy to sketch on a whiteboard.   The problems start when you attempt to implement your ideas, and then you can find yourself with a seemingly insurmountable sackful of (human) objections and (process) hurdles.

### What’s an automation engineer to do?
In most of the companies I’ve brought some automation into, everyone buys into automation except the Security guys.   The security shutters come down when a requirement for ssh or sudo is mentioned.

But hang on a wee minute here….  Both ssh and sudo (correctly configured) are perfectly secure, way more secure than an stale/known database password or a ‘p@ssword123’ (yes, really!).    A re-evaluation of the risk landscape is required if we are blocked because Security perceive ssh/sudo to be an inherently insecure means of change deployment.

Our company, our developers and our infrastructure are committed to becoming more Agile, the question for Security is what are you doing to become more Agile?

Of course we can turn this around, and offer the automation toolset to the Security guys to help them on their Agile journey.   Just think how much more effective security policy could be if it grew with product development, if it responded to changes as an integral component of the deployment pipeline.

We shouldn’t be viewing Security as distinct from Agile/DevOps, we should be helping those guys join the party.   We should be feeding the hand that bites us!

### Example
Let’s put a Security hat on, and walk in their shoes, while we ponder how hard password rotations are.

_Rotating application layer passwords is hard.   Not because changing a password in some database is difficult for these DevOps guys, it’s often only a single command.   The hard part is getting them to actually spend the time doing it.   DevOps don’t seem to understand that all these change requests and procedures are necessary and worthwhile, to ensure they don’t make a mistake._

_If the DevOps guys could stop hard-coding passwords into playbooks, it’s such a security violation.   Though at least it’s an improvement over the others who have to login and manually create/update passwords, that’s really quite 20th century._

### DevOps + Sec?
So if we merge the DevOps thinking with the Sec thinking, we can state the DevSecOps approach to application password rotation:

1. Humans create poor quality passwords, let’s generate them automatically
2. Continuous deployment of password rotations would be ideal
4. An automated task would allow increased password rotation frequency
5. An automated task can be tested, and will never go beyond its scope 
6. Storing the password in a shared-secret vault is our break glass
7. Integrating with AD would be great, allowing seamless access control

What’s a good quality approach to ticking off all of the above points?

A full-fat automation approach of course.   The image shows running a password rotation as an authenticated user.   User _ian_ being authorised to rotate passwords, and user _dervla_ not being authorised.

![Authentication rotate password playbook examples](/images/rotate-password-play_800x600.png)

### Example Code
Our context for this example is a tiered Java application with a mysql backend.   I’ll run through password rotation of the backend mysql root password, but the same principle applies to the application node db credentials.

We have an existing HashiCorp Vault instance available (link to Github repo below).   HashiVault gives us the capability to use LDAP as an authentication backend, so we can point it at our Active Directory and leverage existing users and groups to provide access control.   

This means the account initiating the password rotation task supplies their AD credentials which Ansible uses to authenticate against HashiVault.   We capture credentials using vars_prompt in the playbook

```
# file: rotate password playbook
  vars_prompt:
	- name: "username"
	  prompt: "your AD login: "
	  private: no

	- name: "password"
	  prompt: "password: "
	  private: yes
```

The credentials are used to authenticate against HashiVault, so the playbook fails if the account has no access.   In this case access to the password in vault is locked down to the LDAP/AD group ‘production’.

```
# commands: enabling HashiVault ldap authentication
vault auth-enable ldap
vault write auth/ldap/config url="ldap://ldapserver" \
	userdn="ou=people,ou=development,dc=example,dc=com" \
	groupdn="ou=groups,ou=development,dc=example,dc=com" \
	binddn="cn=Manager,dc=example,dc=com" \
	bindpass='password' \
	starttls=false
vault write auth/ldap/groups/production policies=production
vault write auth/ldap/groups/developer policies=developer
```

With the above authentication configuration, we have an Ansible task to authentice against HashiVault and retrieve a token to be able to read the existing stored mysql password.

```
# file: roles/mysql-rotate-pw/tasks/main.yml
- name: "Authenticate ({% raw %}{{ username }}{% endraw %}) against HashiVault"
  uri:
	url: "http://vault:8200/v1/auth/ldap/login/{% raw %}{{ username }}{% endraw %}"
	method: POST
	body_format: json
	body: '{"password": "{% raw %}{{ password }}{% endraw %}" }'
	return_content: yes
	status_code: 200
  register: auth_request_content
  delegate_to: localhost

	# the vault_user_token is used for subsequent authentication to
	# HashiVault, we set_fact here just to make for easier reading.
- set_fact:
	vault_user_token: "{% raw %}{{ auth_request_content.json.auth.client_token }}{% endraw %}"
```

With successful authentication the vault user token is then used to access the particular shared secret.   This is controlled by the HashiVault policy ‘production’ enabling write access to the dbnode path.

```
# file: roles/mysql-rotate-pw/tasks/main.yml
- name: Read existing mysql password from the HashiVault
  uri:
  	url: "http://vault:8200/v1/secret/prod/apps/communote/dbnode"
  	method: GET
	  HEADER_X-Vault-Token: "{% raw %}{{ vault_user_token }}{% endraw %}"
  	return_content: yes
  	status_code: 200
  register: vault_content
  delegate_to: localhost
```

Some backends in HashiVault can auto generate passwords on a read access, but here I show password creation for the sake of clarity.

```
# file: roles/mysql-rotate-pw/tasks/main.yml 
- name: Generate new random password for mysql root user
  shell: </dev/urandom tr -dc '1234567890qwertyuiopQWERTYUIOPasdfghjklASDFGHJKLzxcvbnmZXCVBNM' | head -c20; echo
  register: rand_pw_string
  delegate_to: localhost
```

And using the retrieved (existing) password we can update the mysql instance with the new password.   Again, I am using this CLI method simply for clarity.

```
# file: roles/mysql-rotate-pw/tasks/main.yml
- name: Update mysql with the new root password
  command: "mysql -u root -p{% raw %}{{ vault_content.json.data.mysqlrootpw }}{% endraw %} -e \"ALTER USER 'root'@'localhost' IDENTIFIED BY '{% raw %}{{ rand_pw_string.stdout }}{% endraw %}'\""
```

Finally we update the password stored in HashiVault.

```
# file: roles/mysql-rotate-pw/tasks/main.yml
- name: Store the new root password back into HashiVault
  uri:
  	url: "http://vault:8200/v1/secret/prod/apps/communote/dbnode"
    method: POST
    HEADER_X-Vault-Token: "{% raw %}{{ vault_user_token }}{% endraw %}"
  	body_format: json
  	body: '{ "mysqlrootpw": "{% raw %}{{ rand_pw_string.stdout }}{% endraw %}" }'
    status_code: 204
  delegate_to: localhost
```

Using our break glass account, for this example, we can demonstrate reading the secret directly from HashiVault.

```
vaultnode $ vault auth -method=ldap username=sec_breakglass
Password (will be hidden): 
Successfully authenticated! You are now logged in.
token: 0425f12f-ad10-811f-ec96-04a6596c5e25
token_policies: [default production]
vaultnode $ vault read secret/prod/apps/communote/dbnode
Key             	Value
---             	-----
mysqlrootpw     	ol4bLyVmcLfvvRd5DdVZ
```

### How we improved our lot
We have achieved all of our seven DevSecOps requirements.   In a practical sense we’ve also improved the security of the infrastructure.   By simply making password rotation practically possible with both high frequency and on a large scale.

I hope this example is useful, in terms of reframing the DevOps approach.  For those of us in large environments, I think we need to consider whether DevOps is discriminatory.   DevSecOps is more inclusive, and a more realistic philosophy to helping companies move further along the Agile/Automation path.

[full code on Github](https://github.com/thisdougb/ansible-hashicorp-rotate-password)

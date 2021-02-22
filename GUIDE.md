Ansible Coding Guide
=======================

## Introduction
The purpose of this document is to give an in-depth explanation of
less obvious points of Ansible development, tips and tricks. Most, if
not all points in this document deserve a mention in the best
practices/coding standards document, but the explanation in that document is
usually brief due to its nature and could be hard to understand for
someone less familiar with Ansible (or Jinja2).

## Lazy evaluation of variables
This is a surprising, sometimes useful, sometimes not [1], feature of
Jinja2 template expansions in variables in Ansible. Ansible expands
the templates in the variable value when the variable is used, not
when it is defined, i.e. variables are evaluated lazily [2]. So one
can do this:
```yaml
vars:
  foo: "{{ bar }}"
tasks:
  - set_fact:
      bar: quux
  - name: use foo
    somemodule:
      someparameter: "{{ foo }}"
```

`foo` is defined before the tasks even start, but can refer to a value
which gets defined only during the execution of tasks! So one can use
a subsequently defined value in a previous variable declaration and
things continue to work.

An example of this feature is
https://github.com/linux-system-roles/storage/pull/81 : when defining
`unused_disk_subfact`, the variable `unused_disks` is used. But this
variable is not available yet. It is set by
`get_unused_disk.yml`. Still, it works, because `unused_disk_subfact`
is actually expanded only after the call to `get_unused_disk.yml`, and
`unused_disks` is already defined here.

Another consequence is that one variable defined in `vars:` can refer
to another variable defined in the same `vars:` block, like here:
```yaml
  vars:
    too_large_size: '{{ (unused_disk_subfact.sectors|int + 1) *
                        unused_disk_subfact.sectorsize|int }}'
    unused_disk_subfact: '{{ ansible_devices[unused_disks[0]] }}'
```
At the point when `too_large_size` will be used, `unused_disk_subfact`
will be already known, and the expansion of `too_large_size` will work
properly.

Another place where this is used is here:
https://github.com/linux-system-roles/timesync/blob/9050dd95314406236bc8869eac2307a6fa932edc/vars/main.yml#L1
Role variables are loaded when the role starts and
`ansible_facts.packages` is not defined yet (it is not among facts
collected by default, we collect it during the role execution). But
this is not a problem when defining `__timesync_chrony_version`,
because the template does not get expanded at this moment - it gets
expanded only when `__timesync_chrony_version` is used, and if it is
after we collect the fact, everything is fine.

(Beware though, `set_facts` expands variables, because it is like a
module call. So in `set_fact` you can not refer to variables whose
values contain templates referring to yet undefined variables. In this
way `set_fact` serves as a checkpoint that expands everything that's
given to it and is not expanded yet. This can be useful, too bad that
facts are global and can not be unset.)

## Setting variables on `roles`/`import_role` statement
Variables set on the `roles` or `import_role` statement appear like
parameters, but actually they are made accessible to the whole
playbook run, just like role variables set in `vars/` - therefore can
influence further role invocations.

Example:
```
- hosts: localhost
  tasks:
    - name: show var1 before
      debug:
        var: var1
    - name: import role without a role var
      import_role:
        name: emptyrole
      vars:
        var1: set!
    - name: show var1 after
      debug:
        var: var1
```
Equivalent example:
```yaml
- hosts: localhost
  roles:
    - name: emptyrole
      vars:
        var1: set!
  pre_tasks:
    - name: show var1 before
      debug:
        var: var1
  tasks:
    - name: show var1 after
      debug:
        var: var1
```
`emptyrole` is an empty role created by
`ansible-galaxy init emptyrole`.

Running both examples show that `var1` is set both in the first
(before calling the role) and 
second task (after calling the role). This can be unexpected, as other instances of setting
`vars:` on a task (including `import_tasks`) keep the variables local
to the task, so the variable would be unset both before and after the
task.
See
https://github.com/ansible/ansible/issues/66714
https://github.com/ansible/ansible/issues/55942
https://github.com/ansible/ansible/issues/50278#issuecomment-450901131

For this reason, either use `include_role`, which does not suffer from
the problem, or wrap `import_role` in a `block:` and set the variables
on the `block:`.


[1] In a role, you can not pass a variable containing a reference to
something like `role_name` to another role and use it to reference to
the calling role, because the variable will only get expanded in the
called role to something else - the variable represents the currently
executing role. https://github.com/ansible/ansible/issues/10374

[2] Another well-known language with this style of variable expansion
is `make`:
https://www.gnu.org/software/make/manual/html_node/Flavors.html

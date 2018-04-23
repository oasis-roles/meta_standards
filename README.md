This document is intended to be a standards document for the Ansible roles contained in the
Oasis Roles repositories.

Basics
======

* Every repository in the oasis-roles namespace should be a valid Ansible Galaxy compatible role
  with the exception of any whose names begin with "meta\_", such as this one.
* New roles should be initiated in line with the skeleton directory, which has standard boilerplate
  code for a Galaxy-compatible Ansible role and some enforcement around these standards

Naming Things
=============

* All YAML or Python files, variables, arguments, repositories, and other such names should follow
  standard Python naming conventions of being in snake\_case\_naming\_schemes.
* Names should be menmonic and descriptive and not strive to shorten more than necessary. Systems
  support long identifier names, so use them to be descriptive
* All defaults and all arguments to a role should have a name that begins with the role name to help
  avoid collision with other names. Avoid names like `packages` in favor of a name like `foo_role_packages`.

Syntax
======

* All roles need to, minimally, pass a basic ansible-playbook syntax check run
* All task arguments should be spelled out in YAML style and not use `key=value` type of arguments
* All YAML files need to pass standard yamllint syntax with the modifications listed in tests/yamllint.yml
  in the meta\_skeleton role. These modifications are minimal: document starter characters (the initial
  `---` string at the top of a file) should not be used, and it is not necessary to start every comment
  with a space. Most comments should start with a space, but no space is allowed when a comment is
  documenting an optional variable with its default value.
* Although it is not expressly forbidden, comments in playbooks should be avoided when possible. The task
  `name` value should be descriptive enough to tell what a task does. Variables should be well commented in
  the `defaults` and `vars` directories and should, therefore, not need explanation in the playbooks
  themselves.

Ansible Best Practices
======================

* Avoid the use of `when: foo_result is changed` whenever possible. Use handlers, and, if necessary, handler
  chains to achieve this same result. Exceptions are permitted but they should be avoided when possible
* Use the various include/import statements in Ansible when doing so can lead to simplified code and a
  reduction of repetition. This is the closest that Ansible comes to callable sub-routines, so use judgment
  about callable routines to know when to similarly include a sub playbook. Some examples of good times
  to do so are
** When a set of multiple commands share a single `when` conditional
** When a set of multiple commands are being looped together over a list of items
** When a single large role is doing many complicated tasks and cannot easily be broken into multiple roles,
   but the process proceeds in multiple related stages
* Avoid calling the `package` module iteratively with the `{{ item }}` argument, as this is impressively
  more slow than calling it with the line `name: "{{ foo_packages | join(',') }}"`. The same can go for
  many other packages that can be given an entire list of items all at once.

Vars vs Defaults
----------------
* Avoid embedding large lists or "magic values" directly into the playbook. Such static lists should be
  placed into the `vars/main.yml` file and named appropriately
* Every argument accepted from outside of the role should be given a default value in `defaults/main.yml`.
  This allows a single place for users to look to see what inputs are expected. Document this file
  copiously
* Use the `defaults/main.yml` file in order to avoid use of the default Jinja2 filter within a playbook.
  Using the default filter is fine for optional keys on a dictionary, but the variable itself should be
  defined in `defaults/main.yml` so that it can have documentation written about it there and so that all
  arguments can easily be located and identified.
* Avoid giving default values in `vars/main.yml` as such values are very high in the precedence order and
  are difficult for users and consumers of a role to override.
* As an example, if a role requires a large number of packages to install, but could also accept a list of
  additional packages, then the required packages should be placed in `vars/main.yml` with a name such as
  `foo_role_packages`, and the extra packages should be passed in a variable named `foo_role_extra_packages`,
  which should default to an empty array in `defaults/main.yml` and be documented as such.

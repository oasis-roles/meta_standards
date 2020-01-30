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
* Names should be mnemonic and descriptive and not strive to shorten more than necessary. Systems
  support long identifier names, so use them to be descriptive
* All defaults and all arguments to a role should have a name that begins with the role name to help
  avoid collision with other names. Avoid names like `packages` in favor of a name like `foo_role_packages`.

YAML Syntax
===========

* All roles need to, minimally, pass a basic ansible-playbook syntax check run
* All task arguments should be spelled out in YAML style and not use `key=value` type of arguments
* All YAML files need to pass standard yamllint syntax with the modifications listed in tests/yamllint.yml
  in the meta\_skeleton role. These modifications are minimal: document starter characters (the initial
  `---` string at the top of a file) should not be used, and it is not necessary to start every comment
  with a space. Most comments should start with a space, but no space is allowed when a comment is
  documenting an optional variable with its default value.
  * As a result of being able to pass basic YAML lint, avoid the use of `True` and `False` for boolean values
   in playbooks. These values are sometimes used because they are the words Python uses. However, they are
   improper YAML and will be treated as either strigns or as booleans but generating a warning depending on
   the particular YAML implementation.
   * Do not use the Ansible-specific `yes` and `no` as boolean values in YAML as these are completely
   custom extensions used by Ansible and are not part of the YAML spec.
* Although it is not expressly forbidden, comments in playbooks should be avoided when possible. The task
  `name` value should be descriptive enough to tell what a task does. Variables should be well commented in
  the `defaults` and `vars` directories and should, therefore, not need explanation in the playbooks
  themselves.

Jinja2 Syntax
=============

* All Jinja2 template points should have a single space separating the template markers from the variable
  name inside. For instance, always write it as `{{ variable_name_here }}`. The same goes if the value is
  an expression. `{{ variable_name | default('hiya, doc') }}`

Ansible Best Practices
======================

* All tasks should be idempotent, with notable and rare exceptions such as the
  [OASIS reboot role](https://github.com/oasis-roles/reboot).
* Avoid the use of `when: foo_result is changed` whenever possible. Use
  handlers, and, if necessary, handler
  chains to achieve this same result. Exceptions are permitted but they should be avoided when possible
* Use the various include/import statements in Ansible when doing so can lead to simplified code and a
  reduction of repetition. This is the closest that Ansible comes to callable sub-routines, so use judgment
  about callable routines to know when to similarly include a sub playbook. Some examples of good times
  to do so are
  * When a set of multiple commands share a single `when` conditional
  * When a set of multiple commands are being looped together over a list of items
  * When a single large role is doing many complicated tasks and cannot easily be broken into multiple roles,
   but the process proceeds in multiple related stages
* Avoid calling the `package` module iteratively with the `{{ item }}` argument, as this is impressively
  more slow than calling it with the line `name: "{{ foo_packages }}"`.  The same can go for many other
  modules that can be given an entire list of items all at once.
* Use meta modules when possible. Instead of using the `upstart` and `systemd` modules, use the `service`
  module when at all possible. Similarly for package management, use `package` isntead of `yum` or `dnf` or
  similar. This will allow our playbooks to run on the widest selection of operating systems possible without
  having to modify any more tasks than is necessary.
* Avoid excessive use of the `lineinefile` module. Slight miscalculations in how it is used can lead to a loss
  of idempotence. Additionally, modifying config files with it can become arcane and difficult to read,
  especially for someone not familiar with the file in question. If the file is not in a form that can be
  modified with a module (e.g. the `ini` module) then default to using the `template` module. This both makes
  it easier to see what is being configured and to add additional, more complex options in the future.
* Avoid the use of `lineinfile` wherever that might be feasible. Try editing files directly using other built
  in modules (e.g. `ini_file`) or reading and parsing. If you are modifying more than a tiny number of lines
  or in a manner more than trivially complex, try leveraging the `template` module, instead. This will allow
  the entire structure of the file to be seen by later users and maintainers.
* Limit use of the `copy` module to copying remote files and to uploading binary blobs. For all other file
  pushes, use the `template` module. Even if there is nothing in the file that is being templated at the
  current moment, having the file handled by the `template` module now makes adding that functionality much
  simpler than if the file is initially handled by the `copy` and then needs to be moved before it can be
  edited.
* When using the `template` module, refrain from appending `.j2` to the file name. This alters the syntax
  highlighting in most editors and will obscure the benefits of highlighting for the particular file type or
  filename. Anything under the `templates` directory of a role is assumed to be treated as a Jinja 2 template,
  so adding the `.j2` extension is redundant information that is not helpful.
* Keep filenames and templates as close to the name on the destination system as possible. This will help with
  both editor highlighting as well as identifying source and destination versions of the file at a glance.
  Avoid duplicating the remote full path in the role directory, however, as that creates unnecessary depth in
  the file tree for the role. Grouping sets of similar files into a subdirectory of `templates` is allowable,
  but avoid unnecessary depth to the hierarchy.

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

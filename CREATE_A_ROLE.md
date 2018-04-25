A basic guide for how to create a new role in oasis-roles

Setup environment
=================

The following Python tools should be installed in either a virtualenv or in your system or elsewise
should be on your system path:

* ansible
* ansible-lint
* yamllint
* flake8
* molecule

Create Role
===========

* `mkdir -p oasis/roles && cd oasis/roles`
* `git clone git@github.com:oasis-roles/meta_skeleton.git`
* `ansible-galaxy init --role-skeleton=meta_skeleton my_role_name`
* `cd my_role_name`
* Edit README.md, meta/main.yml, and write your code

Setup CI Services
=================

* Create repo in GitHub
* Create Travis CI job for new repo
* Add GH remote and push code
* `ansible-galaxy import oasis-roles my_role_name` - adds repo to global Galaxy index
* `ansible-galaxy setup travis oasis-roles my_role_name [Travis CI API key]` - adds auto-updating Galaxy index
  when Travis CI jobs are completed

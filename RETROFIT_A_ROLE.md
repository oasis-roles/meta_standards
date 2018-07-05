A brief guide on how to retrofit a role with CI.

TL;DR
=====

Please, don't make me read all of that down there!

Create Sceanrio
---------------
`molecule init scenario -r my_role -s my_scenario`

Configure Scenario Options
--------------------------
`$EDITOR molecule/my_scenario/molecule.yml` to edit Molecule options
`$EDITOR molecule/my_scenario/converge.yml` to set values for test defaults in the
run
`$EDITOR molecule/my_scenario/[create|destroy].yml` to update driver options

Run Whole Scenario
------------------
`molecule test -s my_scenario`

What follows is additional information about these and other relevant topics

Create Scenarios
================

Each set of tests in Molecule is dubbed a "scenario" and lives in its
own directory underneath of the "molecule" directory. By convention, the name of
the scenario and the name of the directory it is housed in will match, but there
is no technical requirement for this. (The name is actually specified in the
scenario's metadata, and Molecule will search for all files that match the glob
moelcule/\*/molecule.yml for the scenario names.) As a matter of principle, the
convention of keeping the scenario name matching the molecule sub-directory is
a good pattern to maintain. Any deviation from this should only be to name a
scenario "default". When the molecule executable is invoked without a scenario
name, it will look for a scenario named "default" in its glob and execute on
that.

Molecule has a templating system that supports a good set of the existing
Ansible provisioners already. At the least it supports provisioners for GCE,
AWS, Azure, OpenStack, and (local) Docker. The template will create a default
scenario that includes provisioning, teardown, prepare, and converge stages on
the selected system provider. Any of these files can be customized, and the ones
that are included in the default OASIS scenarios are already customized in ways
that match the default OASIS use cases.

First Scenario (Optional)
=========================
If a role exists and has not yet been given any Molecule scenarios yet, then the
process is slightly different than just adding new scenarios to an existing
Moelcule directory. At least, in most cases this will be true. A new scenario is
created by issuing the following command:

`molecule init scenario --role-name [role]`

The `role` argument should be the same as the folder name that the role exists
in. Once this command is run, a new directory called "molecule" will be created
(if that directory already exists, then a new directory will be created inside
of it). Inside of the "molecule" directory will be a child directory named
"default". This directory will be populated with a number of Ansible playbooks,
a directory named "tests", and files named "molecule.yml", "INSTALL.rst", and
"Dockerfile.j2".

The `molecule.yml` file is the metadata file that drives the scenario. Inside of
that you can find the name of the scenario that was created, information about
the driver that will be used (e.g. OpenStack, AWS, Docker), and other options.
Many parts of a Molecule run can be customized from here. This scenario should
be able to run without any modifications by typing

`molecule test`

in the command line. Provided, that is, that your role can be run without any
dependencies or variables being set.

Named Scenarios
===============

By default, the first scenario created above is named "default" and is placed in
the directory "molecule/default". While this can be good for a basic syntax
check, lint check, and the like the name is not very informative and does not
convey much information about what a tester will need in order to run the tests
by themselves. If the default scenario is configured to run outside of Docker,
then the user will need credentials to the test environment configured, etc.

Most scenarios, and all of them that are not the default, will need a name
passed in at creation time. Also, if they are to be executed in any provider
other than Docker, that will need to be set at creation time. This can be
handled thusly:

`molecule init scenario --role-name [role] --driver-name [ec2|openstack|...]
--scenario-name [my_scenario_name]`

This will create a new scenario in the folder "molecule/my\_scenario\_name".
By convention, in the OASIS project, we name all our scenarios after the driver
that backs them. This can be extended if thare are multiple test scenarios for
the same driver, then a distinguishing name can be added in snake case after the
driver name (e.g. "openstack\_condition\_foo").

Provided you have credentials configured for the driver environment, you should
be able to now execute `molecule test -s my_scenario_name` and the test should
complete just as the default role above completed.

Important Molecule Information
==============================

Molecule documentation is not the most robust. A brief overview of some of the
information can be shared here. Reach out to the OASIS develoeprs if there are
more questions you have, or open issues agains this repository to ask for more
documenation here.

Scenario Name
-------------

Nearly every single command to molecule will require being run within the
context of one of the scenarios. If none is specfied, the scenario named
"default" is sought. If it is not found, Molecule will exit with an error
message inidcating it can't find the scenario to execute. By default, OASIS
roles that follow the "meta\_skeleton" repository do not have a default
scenario. This is not because a default is discouraged, but just because the
nature of the skeleton is such that a good default is hard to predict.

In order to shorten your commands, it is perfectly reasonable to have a default
role that is created. With the exception of the `init` sub-command, every other
command mentioned in these documents probably accepts the "-s" argument to
specify the scenario name. Just because one is not given in a sample version of
the command does not mean it isn't accepted by the command. As always, refer to
the Molecule help pages to be sure.

Actions
------

A full run of a molecule scenario (`moelcule test [-s scenario_name]`) will step
through a number of stages. This can be seen in the output of a run with text
like

``
--> Scenario: 'default'
--> Action: 'destroy'
``

This tells you which action is being taken at which point through the run. Most
of these actions are available as direct subcommands of Moelcule and may be
invoked as such. For instance, to ensure that all resources for a particular
scenario have been torn down, you can run `moelcule destroy [-s scenario name]`.
Likewise for most of the other actions.

This can become particularly helpful if you are running into linting problems
and don't want to wait for a whole VM or Docker container to be spun up. Then
you can just run `molecule lint [-s scenario name]`. Linting is one of the main
reasons that it can be helpful to have a default scenario, as it shortens the
length of this command.

Likewise, if you are iterating on a particular problem with a role, you may not
want to execute `molecule test` over and over, as that will de-provision all
resources when the test fails. Instead, you can run `moelcule create` and then
your testing can be cycled by running `molecule converge` repeatedly until the
role completes properly. At the end, you can then run `molecule destroy` to
clean things up.

Most of the actions invoked as part of invoking any other action, including
"test", can be customized. Refer to the Molecule documentation for information
on both what the default values are, and what settings to add to "molecule.yml"
to customize the order to to omit/add steps.

Important Files
---------------

Most scenarios will come with a "create.yml", "prepare.yml", and "destroy.yml"
playbook in their core directories. These are just regular Ansible playbooks.
Each of these playbooks corresponds to an action in the Moelcule test lifecycle.
Predictably, the names of the files match the names of the actions. Create and
destroy map to the process of provisioning and tearing down resources in the
target environment. In the default arrangement, those are the first steps after
linting and the last step taken by Molecule.

The "prepare.yml" file is a general playbook that will be executed against the
provisioned resources. This is a good place to setup the hosts in the state they
are expected to be before the current role gets executed. This playbook will
only be executed once against the target hosts, so this is a fine place to put
any weird configurations or non-idempotent actions that need to be taken to
setup the scenario. A good example here might be a role that tests updating an
application might use the "prepare.yml" file to install the application and get
it running before the role gets tested. A `molecule prepare` can be used to
execute this playbook.

The final playbook included in each scenario is "converge.yml". This file
defines the actual test run for the role. This file can be executed by calling
`molecule converge`. This is a place to put test values for role variables and
other arguments, as well as call supporting roles and anything else that might
be necessary to execute the role in this test. If your role can reasonably be
tested by just running additional Ansible commands, adding them to this file
either as a final play or as `post_actions` is a fine way to make sure that the
role has done its work properly.

Shared Code
===========

As one might expect, it is possible that code might be shareable between
scenarios that have different drivers. For example, the "prepare.yml" file is
a playbook that will be executed exactly one time on each host before the role
being tested. It is very feasible that this will be the same between all runs
regardless of driver. If this is the case, the path to each such file can be
customized in the "molecule.yml" file. See various examples within the OASIS
namespace for how to do that, including the "meat\_skeleton" role, which places
several such files into the folder "molecule/shared".

One example of code that almost certainly should be shared between all of the
scenarios is the code for configuring the lint stage. The "meta\_skeleton"
repository has the option `lint.options.config-file` set to read from a shared
config file named "tests/yamllint.yml" that lives relative to the root of the
role. For customizing the paths to the playbooks in the scenario, those paths
must be relative to the current scenario directory.

Testinfra Tests
===============

For any types of tests that are not or should not be written directly into the
"converge.yml" playbook, tests can be written in Python using the Test Infra
framework. An example test is included in the molecule scenario scaffolding
directories. Others can be found in some of the existing OASIS roles. Other
roles can be sufficiently tested from within Ansible (e.g. it is not necessary
to test with Python that a system repository is properly configured. The
converge.yml file can include post\_actions tasks that install something from
the newly configured test repository to verify the role has behaved properly).

Testinfra exists and is part of the Molecule environment. However, where easy,
keep tests in the Ansible playbooks so they are more accessible to people
reading and editing the tests after you.

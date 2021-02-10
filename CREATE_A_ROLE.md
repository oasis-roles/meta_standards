A basic guide for how to create a new role in oasis-roles

# Setup environment #

* Install [podman](https://podman.io/) using your system package manager.  This
  example is for Fedora:

  `sudo dnf install podman-docker`

* Create a virtualenv.
  [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/install.html#basic-installation)
  is recommended.

  `mkvirtualenv oasis`

* Install Python dependencies.

  `pip3 install ansible tox-ansible pytest-molecule molecule-openstack molecule-podman molecule-docker os-cloud-config`

* Install the OASIS
  [ansible_collection_system](https://github.com/oasis-roles/ansible_collection_system)
  collection via Galaxy to use the
  [molecule_openstack_ci](https://github.com/oasis-roles/ansible_collection_system/tree/master/roles/molecule_openstack_ci)
  role for OpenStack provisioning.

  `ansible-galaxy collection install oasis_roles.system`

* Clone an existing collection.  In this example we'll be using the
  [ansible_collection_system](https://github.com/oasis-roles/ansible_collection_system)
  collection.  Git submodules are used to bring in shared repo dependencies.

  `git clone --recurse-submodules git@github.com:oasis-roles/ansible_collection_system.git`

* Clone the OASIS [meta_skeleton](https://github.com/oasis-roles/meta_skeleton)
  role template.  We delete the `.git` directory of this repo due to Ansible
  bug [#71977](https://github.com/ansible/ansible/issues/71977).

  `git clone git@github.com:oasis-roles/meta_skeleton.git && rm -rfv ./meta_skeleton/.git`

* To use Subscription Manager via the OASIS
  [rhsm](https://github.com/oasis-roles/ansible_collection_system/tree/master/roles/rhsm)
  role, add the following variables to your environment and change the values
  according to your Subscription Manager configuration.

  ```
  export OASIS_RHSM_USERNAME=myusername
  export OASIS_RHSM_PASSWORD=mypassword
  export OASIS_RHSM_SERVER_HOSTNAME=subscription.rhn.stage.redhat.com
  export OASIS_RHSM_POOL_IDS=['mypoolid']
  ```

* To use OpenStack to run scenarios, create the following directory.

  `mkdir -p ~/.config/openstack`

* Log into the web UI of your OpenStack instance, and from the UI select the
  following options to download your `clouds.yaml` file to the
  `~/.config/openstack` directory.

  _Project -> API Access -> Download OpenStack RC File -> OpenStack clouds.yaml File_

# Create a new role #

* Change directory to the collection repo.

  `cd ansible_collection_system`

* Create a new role directory using the template.

  `ansible-galaxy init --role-skeleton=../meta_skeleton/ --init-path=./roles myrole`

* Edit the `README.md` and `meta/main.yml` files in the new role directory with
  the appropriate details.

# Working with [tox-ansible](https://github.com/ansible-community/tox-ansible) and [Molecule](https://molecule.readthedocs.io/en/latest/) #

* From the root of a collection, you may list existing
  tox environments.

  `tox -l`

* Run tox on one of the environments from the output above to test your
  environment.  From the `tox -l` output, the suffix `-docker` indicates the
  Molecule scenario name.  Any given role may have multiple scenarios, each
  with their own provisioner (typically Docker or OpenStack).  Note that Docker
  scenarios are mostly used for linting, as many of the OASIS roles require VMs
  to execute and test the actual automation code.

  `tox -e roles-package_updater-docker -- -d podman`

* If you're debugging a failure and would like to leave provisioned instances
  up to check their error state, run the following.

  `tox -e roles-package_updater-docker -- -d podman --destroy never`

* To get debugging output from Ansible (`-v`) and Molecule (`--debug`), you
  must run Molecule directly since tox-ansible is not able to pass debugging
  options down the chain.

  `cd roles/myrole`

  `molecule -v --debug -c ../../tests/molecule.yml test -s docker -d podman`

# Setup CI Services #

TODO:  Document CI with GitHub Actions

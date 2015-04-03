Contributing
============

Installing Akanda Locally for Development
-----------------------------------------

Akanda's own `continuous integration <http://ci.akanda.io>`_ is open source
`(github.com/akanda/akanda-ci) <https://github.com/akanda/akanda-ci>`_, and
includes `Ansible <http://ansibleworks.com>`_ playbooks which can be used to
spin up the Akanda platform with a `devstack
<http://docs.openstack.org/developer/devstack/>`_ installation::

    $ pip install ansible
    $ ansible-playbook -vv playbooks/ansible-devstack.yml -e "branch=stable/juno"

Submitting Code Upstream
------------------------

All of Akanda's code is 100% open-source and is hosted `on GitHub
<http://github.com/akanda/>`_.  Pull requests are welcome!

*******
Install
*******

This set of playbooks have specific dependencies on Ansible due to the modules
being used.

Requirements
============

* Ansible 2.7 or >
* Docker Engine
* docker

Install OS dependencies on CentOS 7

.. code-block:: bash

  $ sudo yum install -y epel-release
  $ sudo yum install -y gcc python3-pip python-devel openssl-devel
  # If installing Molecule from source.
  $ sudo yum install libffi-devel git

Install OS dependencies on Ubuntu 16.x

.. code-block:: bash

  $ sudo apt-get update
  $ sudo apt-get install -y python3-pip libssl-dev docker-engine
  # If installing Molecule from source.
  $ sudo apt-get install -y libffi-dev git

Install OS dependencies on Mac OS

.. code-block:: bash

  $ brew install python3
  $ brew install git

Install using pip:

.. code-block:: bash

  $ sudo pip install ansible
  $ sudo pip install docker
  $ sudo pip install molecule[docker] --pre

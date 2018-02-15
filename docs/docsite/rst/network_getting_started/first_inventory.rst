Network Getting Started: Working with Inventory
===============================================

A fully-featured inventory file can serve as the source of truth for your network. Using an inventory file, a single playbook can maintain hundreds of network devices with a single command. This page shows you how to build an inventory file, step by step.

First, group your devices by OS and/or by function. You can group groups using the syntax ``metagroupname:children`` and listing groups as members of the metagroup. In this tiny example data center, the group ``network`` includes all leafs and all spines; the group ``datacenter`` includes all network devices plus all webservers.

.. code-block:: yaml

   [leafs]
   leaf01
   leaf02

   [spines]
   spine01
   spine02

   [network:children]
   leafs
   spines

   [webservers]
   webserver01
   webserver02

   [datacenter:children]
   leafs
   spines
   webservers


Next, you can set values for many of the variables you needed in your first Ansible command in the inventory, so you can skip them in the ansible-playbook command. In this example, the inventory includes each network device's IP, OS, and SSH user. If your network devices are only accessible by IP, you must add the IP to the inventory file. If you access your network devices using hostnames, the IP isn't necessary. In an inventory file you **must** use the syntax ``key=value`` for variable values.

.. code-block:: yaml

   [leafs]
   leaf01 ansible_host=10.16.10.11 ansible_network_os=vyos ansible_user=my_vyos_user
   leaf02 ansible_host=10.16.10.12 ansible_network_os=vyos ansible_user=my_vyos_user

   [spines]
   spine01 ansible_host=10.16.10.13 ansible_network_os=vyos ansible_user=my_vyos_user
   spine02 ansible_host=10.16.10.14 ansible_network_os=vyos ansible_user=my_vyos_user

   [network:children]
   leafs
   spines

   [servers]
   server01 ansible_host=10.16.10.15 ansible_user=my_server_user
   server02 ansible_host=10.16.10.16 ansible_user=my_server_user

   [datacenter:children]
   leafs
   spines
   servers

When devices in a group share the same variable values, such as OS or SSH user, you can reduce duplication and simplify maintenance by consolidating these into group variables:

.. code-block:: yaml

   [leafs]
   leaf01 ansible_host=10.16.10.11
   leaf02 ansible_host=10.16.10.12

   [leafs:vars]
   ansible_network_os=vyos
   ansible_user=my_vyos_user

   [spines]
   spine01 ansible_host=10.16.10.13
   spine02 ansible_host=10.16.10.14

   [spines:vars]
   ansible_network_os=vyos
   ansible_user=my_vyos_user

   [network:children]
   leafs
   spines

   [servers]
   server01 ansible_host=10.16.10.15
   server02 ansible_host=10.16.10.16

   [datacenter:children]
   leafs
   spines
   servers

As your inventory grows, you may want to group devices by platform and move shared variables out of the main inventory file into a set of group variable files. This reduces duplication further and sets the stage for managing devices on multiple platforms in a single inventory file. The directory tree for this setup looks like this:

.. code-block:: console

   .
   ├── first_playbook.yml
   ├── inventory
   ├── group_vars
       └── vyos.yml

with inventory:

.. code-block:: yaml

   [vyos_leafs]
   leaf01 ansible_host=10.16.10.11
   leaf02 ansible_host=10.16.10.12

   [vyos_spines]
   spine01 ansible_host=10.16.10.13
   spine02 ansible_host=10.16.10.14

   [vyos:children]
   vyos_leafs
   vyos_spines

   [network:children]
   vyos

   [servers]
   server01 ansible_host=10.16.10.15
   server02 ansible_host=10.16.10.16

   [datacenter:children]
   vyos
   servers

and group_vars/vyos.yml:

.. code-block:: yaml

   ansible_connection: network_cli
   ansible_network_os: vyos
   ansible_user: my_vyos_user

With this setup, you can run first_playbook.yml with only two flags:

.. code-block:: bash

   ansible-playbook -i inventory -k first_playbook.yml

The ``-k`` flag means Ansible will prompt you for SSH passwords. Alternatively, you can store SSH and other device passwords securely in your group_vars files with ``ansible-vault``.

Protecting Sensitive Data with ansible-vault 
```````````````````````````````````````````````````````````````

The ``ansible-vault`` command provides encryption for files and/or strings like passwords. First you must create a password for ansible-vault itself. Then you can encrypt dozens of different passwords across your Ansible project. You can access all those secrets with a single password (the ansible-vault password) when you run your playbooks. Here's a simple example.

Create a file and write your password for ansible-vault to it:

.. code-block:: bash

   echo "my-ansible-vault-pw" > ~/my-ansible-vault-pw-file

Encrypt the ssh password for your VyOS network devices, pulling your ansible-vault password from the file you just created:

.. code-block:: bash

   ansible-vault encrypt_string --vault-id my_user@~/my-ansible-vault-pw-file 'VyOS_SSH_password' --name 'ansible_ssh_pass'

If you prefer to type your vault password rather than store it in a file, you can request a prompt:

.. code-block:: bash

   ansible-vault encrypt_string --vault-id my_user@prompt 'VyOS_SSH_password' --name 'ansible_ssh_pass'

and type in the vault password for ``my_user``. 

The ``--vault-id`` flag allows different vault passwords for different users or different levels of access. Note that the user name ``my_user`` appears in the output of the ``ansible-vault`` command:

.. code-block:: bash

   ansible_ssh_pass: !vault |
          $ANSIBLE_VAULT;1.2;AES256;my_user
          66386134653765386232383236303063623663343437643766386435663632343266393064373933
          3661666132363339303639353538316662616638356631650a316338316663666439383138353032
          63393934343937373637306162366265383461316334383132626462656463363630613832313562
          3837646266663835640a313164343535316666653031353763613037656362613535633538386539
          65656439626166666363323435613131643066353762333232326232323565376635
   Encryption successful

Copy this output into your ``group_vars/vyos.yml`` file, which now looks like this:

.. code-block:: yaml

   ansible_connection: network_cli
   ansible_network_os: vyos
   ansible_user: my_vyos_user
   ansible_ssh_pass: !vault |
          $ANSIBLE_VAULT;1.2;AES256;my_user
          66386134653765386232383236303063623663343437643766386435663632343266393064373933
          3661666132363339303639353538316662616638356631650a316338316663666439383138353032
          63393934343937373637306162366265383461316334383132626462656463363630613832313562
          3837646266663835640a313164343535316666653031353763613037656362613535633538386539
          65656439626166666363323435613131643066353762333232326232323565376635

To run a playbook with this setup, drop the ``-k`` flag and add a flag for your ``vault-id``:

.. code-block:: bash

   ansible-playbook -i inventory --vault-id my_user@~/my-ansible-vault-pw-file first_playbook.yml

Or with a prompt instead of the vault password file:

.. code-block:: bash

   ansible-playbook -i inventory --vault-id my_user@prompt first_playbook.yml


WARNING: Every time you change an ansible-vault password, you must update all files and strings encrypted using that password. If you do not update the encryption, and you cannot access the password used to encrypt a particular file or string, you will not be able to access that file or string. 

For more details on building inventory files, see :doc:`the introduction to inventory<../user_guide/intro_inventory>`; for more details on ansible-vault, see :doc:'the full Ansible Vault documentation<../user_guide/vault>.


Organizing and Your Inventory and Other Ansible Files
```````````````````````````````````````````````````````````````

Ansible expects to find certain files in certain places. As you expand your inventory and create more playbooks, your working Ansible project directory looks like this:

.. code-block:: console

   .
   ├── backup
   │   ├── vyos.example.net_config.2018-02-08@11:10:15
   │   ├── vyos.example.net_config.2018-02-12@08:22:41
   ├── first_playbook.yml
   ├── inventory
   ├── group_vars
   │   ├── vyos.yml
   │   └── eos.yml
   ├── second_playbook.yml
   └── third_playbook.yml


Tracking Changes to Inventory and Playbooks with Git
```````````````````````````````````````````````````````````````

start here in the morning . . .
Network Getting Started: Beyond the Basics
======================================================

This page introduces some concepts that help you manage your Ansible workflow, including roles, directory structure, and source control. Like the Basic Concepts at the beginning of this guide, these intermediate concepts are common to all uses of Ansible.

Beyond Playbooks: Moving Tasks and Variables into Roles
```````````````````````````````````````````````````````````````

Roles are sets of Ansible defaults, files, tasks, templates, variables, and other Ansible components that work together. As you saw on the Working with Playbooks page, moving from a command to a playbook makes it easy to run multiple tasks and repeat the same tasks in the same order. Moving from a playbook to a role makes it even easier to reuse and share your ordered tasks. For more details, see the :doc:`documentation on roles<../user_guide/playbooks_reuse_roles>`. You can also look at :doc:`Ansible Galaxy<../reference_appendices/galaxy>`, which lets you share your roles and use others' roles, either directly or as inspiration.

A Typical Ansible Filetree
```````````````````````````````````````````````````````````````

Ansible expects to find certain files in certain places. As you expand your inventory and create and run more network playbooks, keep your files organized in your working Ansible project directory like this:

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
   ├── roles
   │   ├── static_route
   │   └── system
   ├── second_playbook.yml
   └── third_playbook.yml

The ``backup`` directory and the files in it get created when you run modules like ``vyos_config`` with the ``backup: yes`` parameter.


Tracking Changes to Inventory and Playbooks: Source Control with Git
````````````````````````````````````````````````````````````````````

As you expand your inventory, roles and playbooks, you should place your Ansible projects under source control. We recommend ``git`` for source control. ``git`` provides an audit trail, letting you you track changes, roll back mistakes, view history and share the workload of managing, maintaining and expanding your Ansible ecosystem. There are plenty of tutorials and guides to using ``git`` available.



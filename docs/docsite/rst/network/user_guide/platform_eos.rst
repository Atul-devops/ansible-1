.. _eos_platform_options:

***************************************
EOS Platform Options
***************************************

Arista EOS supports multiple connections. This page offers details on how each connection works in Ansible 2.5 and how to use it. 

.. contents:: Topics

Connections Available
================================================================================

+---------------------------+-----------------------------------------------+---------------------------------------+
|..                         | CLI                                           | eAPI                                  |
+===========================+===============================================+=======================================+
| **Protocol**              |  SSH                                          | HTTP(S)                               |
+---------------------------+-----------------------------------------------+---------------------------------------+
| **Credentials**           | - uses SSH keys / SSH-agent if present        | uses HTTPS certificates if present    |
|                           | - accepts ``-u myuser -k``                    |                                       |
+---------------------------+-----------------------------------------------+---------------------------------------+
| **Indirect Access**       | via a bastion (jump host)                     | via a web proxy                       |
+---------------------------+-----------------------------------------------+---------------------------------------+
| **Connection Settings**   | ``ansible_network_connection: network_cli``   | ``ansible_network_connection: local`` |
|                           |                                               | - requires ``transport: eapi``        |
|                           |                                               | - in the ``provider`` dictionary      |
+---------------------------+-----------------------------------------------+---------------------------------------+
| **Enable Mode**           | supported - use ``become``                    | supported - use ``authorize: yes``    |
| (Privilege Escalation)    |                                               | in the ``provider`` dictionary        |
+---------------------------+-----------------------------------------------+---------------------------------------+
| **Returned Data Format**  | ``stdout[0].``                                | ``stdout[0].messages[0].``            |
+---------------------------+-----------------------------------------------+---------------------------------------+


Using CLI in Ansible 2.5
================================================================================

Example CLI ``group_vars/eos.yml``
----------------------------------

.. code-block:: yaml

   ansible_connection: network_cli
   ansible_network_os: eos
   ansible_user: myuser
   ansible_ssh_pass: !vault...
   ansible_become: yes
   ansible_become_method: enable
   ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q bastion01"'

If you are using SSH keys (including an ssh-agent) you can remove the ``ansible_ssh_pass`` configuration.

If you are accessing your host directly (not through a bastion/jump host) you can remove the ``ansible_ssh_common_args`` configuration.

Example CLI Task
----------------

.. code-block:: yaml

   - name: Backup switch (eos)
     eos_config:
       backup: yes
     register: backup_eos_location
     when: ansible_network_os == 'eos'



Using eAPI in Ansible 2.5
================================================================================

Enabling eAPI
-------------

Before you can use eAPI to connect to a switch, you must enable eAPI. To enable eAPI on a new switch via Ansible, use the ``eos_eapi`` module via the CLI connection. Set up group_vars/eos.yml just like in the CLI example above, then run a playbook task like this:

.. code-block:: yaml

   - name: Enable eAPI
      eos_eapi:
          enable_http: yes
          enable_https: yes
      become: true
      become_method: enable
      when: ansible_network_os == 'eos'

To find out more about the options for enabling HTTP/HTTPS and local http see the :ref:`eos_eapi <eos_eapi>` module documentation.

Once eAPI is enabled, change your ``group_vars/eos.yml`` to use the eAPI connection.

Example eAPI ``group_vars/eos.yml``
-----------------------------------

.. code-block:: yaml

   ansible_connection: local
   ansible_network_os: eos
   ansible_user: myuser
   ansible_pass: !vault | 
   eapi:
     host: "{{ inventory_hostname }}"
     transport: eapi
   proxy_env:
     http_proxy: http://proxy.example.com:8080

If you are accessing your host directly (not through a web proxy) you can remove the ``proxy_env`` configuration.

If you are accessing your host through a web proxy using ``https``, change ``http_proxy`` to ``https_proxy``.


Example eAPI Task
-----------------

.. code-block:: yaml

   - name: Backup switch (eos)
     eos_config:
       backup: yes
       provider: "{{ eapi }}"
     register: backup_eos_location
     environment: "{{ proxy_env }}"
     when: ansible_network_os == 'eos'

In this example two variables defined in ``group_vars`` get passed to the module of the task: 

- the ``eapi`` variable gets passed to the ``provider`` option of the module
- the ``proxy_env`` variable gets passed to the ``environment`` option of the module


.. warning:: 
   Never store passwords in plain text. We recommend using :ref:`Ansible Vault <playbooks_vault>` to encrypt all sensitive variables.

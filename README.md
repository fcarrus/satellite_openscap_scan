satellite_openscap_scan
=========

This role allows Satellite Clients to execute OpenSCAP scannings and report their results to Satellite without installing/configuring Puppet or Ruby.

Requirements
------------

This role is meant to be used on Tower, but of course it works with the plain Ansible Engine too.
You also need Satellite, and some registered hosts.

Role Variables
--------------

Define a Custom Credential on Tower with these three variables (or just use extra vars on the Template):
* satellite_host
* satellite_username
* satellite_password

They will be used to query the Policies to be used during the OpenSCAP scanning.

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: fcarrus.satellite_openscap_scan }

License
-------

GPLv2

Author Information
------------------

Fulvio Carrus <fcarrus@redhat.com> - Red Hat Cloud Consultant


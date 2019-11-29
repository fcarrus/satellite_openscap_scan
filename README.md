satellite_openscap_scan
=========

This role allows Satellite Clients to execute OpenSCAP scannings and report their results to Satellite without installing/configuring Puppet or Ruby.



A little background on OpenSCAP and Satellite
---------------------------------------------

[OpenSCAP] (https://www.open-scap.org) with its `oscap` tool is used to execute a compliance scan on your systems, with rules coming from well defined [policies](https://www.open-scap.org/security-policies/choosing-policy/).

After you scan your systems, the `oscap` tool produces an XML machine-readable report, that when submitted to Satellite, becomes a human-readable html page, where you can learn about all the PASSed and FAILed checks, what they mean and how to fix them.

Satellite uses Puppet to configure the systems so they perform this check on a crontab schedule, like this: 
* It installs the `oscap` tool (openscap-scanner, openscap packages) 
* It installs the `foreman_scap_client` script (rubygem-foreman_scap_client package) - a script to wrap the actual call to the `oscap` tool.
* It configures the `/etc/foreman_scap_client/config.yaml` yaml file with the hostname for the Satellite server and the chosen policy for that particular system.
* It creates the `/etc/cron.d/foreman_scap_client_cron` file, which runs the `foreman_scap_client` script.

Upon running, the `foreman_scap_client` tool (written in Ruby), does the following:
* Retrieves the config from its yaml file
* Downloads the appropriate policy (XML file) from Satellite and caches it locally
* Runs the oscap tool with some flags
* Compresses the results file
* Uploads it to Satellite to the `/api/compliance/arf/:id` endpoint

Upon receiving a report, Satellite makes it available in human-readable form at the Host's page (Compliance tab).

So basically, to run a simple `oscap` command (and a couple of `curl`s), you need to configure the Puppet agent on your systems and have them install Ruby. TBH, I don't want this and neither do many customers.

Here comes Ansible
------------------

Ansible is agentless.  You don't need to install new languages and you already have the Ansible engine on the Satellite machine (and its Capsules).

By using this Ansible Role, the scanned systems only need to install the `oscap` tool itself, from the RHEL repository. 

Upon running, this role does the following:
* Retrieves the policy info for the scanned host, from Satellite
* Downloads the XML file for the policy and the customizations you made for the applied policy (tailoring file)
* Runs the `oscap` command
* Uploads the report to Satellite


How do I start using this role for scanning my hosts?
-------------------------------

_Note_: Red Hat Satellite documentation for the subject is [here](https://access.redhat.com/documentation/en-us/red_hat_satellite/6.6/html/administering_red_hat_satellite/chap-red_hat_satellite-administering_red_hat_satellite-security_compliance_management).

The first thing to do is ensure you already have the SCAP Contents loaded in Satellite.  If you're not sure, run 

```sh
foreman-rake foreman_openscap:bulk_upload:default
```

on the Satellite machine.

In the Satellite GUI, navigate to Hosts -> SCAP Contents. You should see a list of contents. If not, please see the documentation link above.

Now it's time to [choose a policy](https://www.open-scap.org/security-policies/choosing-policy/) for your systems (i.e. [PCI-DSS](https://www.pcisecuritystandards.org/document_library?association=PCI-DSS), suitable for credit card payment systems). Red Hat Satellite already provides some of these policies so that you can choose from one of them, or maybe upload your custom one.

Now navigate to Hosts -> Policies, where you define a Policy.  Click on New Compliance Policy, then:
* Choose *manual* in Deployment Options tab (v6.6 onwards)
* Give it a name in the Policy Attributes tab, (i.e. *RHEL7-PCI-DSS*), and a description if needed.
* In the SCAP Content tab, choose 
  * the SCAP Content (i.e. *Red Hat rhel7 default content*)
  * the XCCDF Profile (i.e. *PCI-DSS v3 Control Baseline for Red Hat Enterprise Linux 7*)
  * the Tailoring file, if you have one.
* In the Schedule tab, choose the cron Period. You can even opt for a custom cron line.
* Select all the applying Locations and Organizations in the next two tabs.
* Select the applying Hostgroups. It's ok if you haven't grouped your hosts, but it's handy if you do it. You may apply this Policy later, host-by-host.

_Note_: If you need to apply the Policy to single hosts, select them from the *Hosts* page and from the *Select Action* choose *Assign Compliance Policy*.  Choose the policy and click Submit.

For every host, the chosen policy can be seen at the `/api/hosts/hostname/enc` endpoint, in the `.classes.foreman_scap_client.policies` json node.

Install this role in Satellite:

```sh
ansible-galaxy install fcarrus.satellite_openscap_scan -p /etc/ansible/roles
```

From the Satellite GUI, go to Configure -> Ansible/Roles. If you don't see the installed role, click the *Import from ...* button.

Clic on the Variables button for the role.  You should define the following variables:

```yaml
satellite_username: myuser
satellite_password: mypassword
satellite_host: satellite.example.com
```



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


---
layout: default
subtitle: "Puppet Enterprise Overview"
canonical: "/pe/latest/overview_about_pe.html"
---


Thank you for choosing Puppet Enterprise (PE), IT automation software that allows system administrators to programmatically provision, configure, and manage servers, network devices, and storage, in a data center or in the cloud.

This user's guide will help you start using Puppet Enterprise 2015.2, and will serve as a reference as you gain more experience. It covers PE-specific features and offers brief introductions to Puppet and the orchestration engine. Use the **navigation at left** to move between the guide's sections and chapters.

> For New Users
> -----
>
> If you've never used PE before and want to evaluate it, you can download PE with ten complimentary licenses and follow the [Puppet Enterprise quick start guide](./quick_start.html). This walkthrough will guide you through creating a small proof-of-concept deployment while demonstrating the core features and workflows of PE.

> For Returning Users
> -----
>
> See the [release notes](./release_notes.html) for detailed information about new features in the PE 2015.2 series.

About Puppet Enterprise
-----

Puppet Enterprise is a comprehensive tool for managing the configuration of enterprise systems. Specifically, PE offers:

* Configuration management tools that let you define a desired state for your infrastructure and then automatically enforce that state.
* A web-based console UI and APIs for analyzing events, managing your nodes and users, and editing resources on the fly.
* Powerful orchestration capabilities.
* An advanced provisioning application called Razor that can deploy bare metal systems.

PE consists of a complete stack of Puppet Labs' technologies, which are automatically installed and connected. [What Gets Installed Where](./install_what_and_where.html#puppet-enterprise-software-components) includes a list of all the major packages that comprise PE 2015.2.

About Puppet
-----

Puppet is the leading open source configuration management tool. It allows system configuration "manifests" to be written in a high-level <abbr title="Domain-Specific Language">DSL</abbr> and can compose modular chunks of configuration to create a machine's unique configuration. PE uses a client/server Puppet deployment, where agent nodes fetch configurations from a central Puppet master.

About Orchestration
-----

PE includes distributed task orchestration features. Nodes managed by PE will listen for commands over a message bus and independently take action when they hear an authorized request. This lets you investigate and command your infrastructure in real time without relying on a central inventory.

About the Puppet Enterprise Console
-----

PE's console is the web-based user interface for managing your systems. The console can:

* Browse and compare resources on your nodes in real time
* Analyze events and reports to help you visualize your infrastructure over time
* Browse inventory data and backed-up file contents from your nodes
* Group similar nodes and control the Puppet classes they receive in their catalogs
* Manage user access, including integration with external user directories

Licensing
-----

PE can be evaluated with a complimentary ten node license. For larger deployments, you will need a commercial per-node license. A license key file will be emailed to you after your purchase, and the Puppet master will look for this key at `/etc/puppetlabs/license.key`. PE will log warnings if the license is expired or exceeded, and you can view the status of your license by running `puppet license` at the command line on the Puppet master.

To purchase a license, please see the [PE pricing page](http://www.puppetlabs.com/puppet/how-to-buy/), or contact Puppet Labs at <sales@puppetlabs.com> or (877) 575-9775. For more information on licensing terms, please see [the licensing FAQ](http://www.puppetlabs.com/licensing-faq/). If you have misplaced or never received your license key, please contact <sales@puppetlabs.com>.

#### License Files in the Console

The licenses menu shows you the number of nodes that are currently active and the number of nodes still available on your current license. If the number of available licenses is exceeded, a warning will be displayed. The number of used licenses is determined by the number of active nodes known to PuppetDB. This is a change from previous behavior in which the number of used licenses was determined based on the number of unrevoked certs known by the CA to determine used licenses. The menu item provides convenient links to purchase and pricing information.

Unused nodes will be deactivated automatically after seven days with no activity (no new facts, catalog, or reports), or you can use `puppet node deactivate` for immediate results. The console will cache license information for some time, so if you have made changes to your license file (e.g. adding or renewing licenses), the changes may not show for up to 24 hours.

* * *

- [Next: Release Notes](./release_notes.html)

# Regenerating certificates: monolithic installs

In some cases, you may find that you need to regenerate the certificates and security credentials \(private and public keys\) generated by PE's built-in certificate authority \(CA\). For example, you may have a Puppet master that you need to move to a different network in your infrastructure, or you may find that you need to regenerate all the certificates and security credentials in your infrastructure due to an unforeseen security vulnerability.

**Related information**  


[Regenerating certificates: split installs](regenerating_certificates_split_installs.md#)

[Regenerate Puppet agent certificates](regenerate_puppet_agent_certificates.md)

[Generate a custom Diffie-Hellman parameter file](generate_custom_dh_parameter_file.md)

## Regenerate certificates in PE: monolithic installs

You can regenerate all certificates in a monolithic PE deployment, including the certificates and keys for the Puppet master, PuppetDB, console, and associated services.

### Before you begin

-   You must be logged in as a root to make these changes.

-   In the following instructions, when `<CERTNAME>` is used, it refers to the agent's certname. To find this value, run `puppet config print certname` before starting.


### About this task

Regenerating your certificates will invalidate all existing authentication tokens. Once the regeneration process is complete, all PE users must generate new authentication tokens.

### Back up certificate directories

If something goes wrong during the regeneration process, you may need to restore these directories so your deployment can stay functional. However, if you needed to regenerate your certs for security reasons and couldn't, you should contact Puppet support as soon as you restore service so we can help you secure your site.

#### About this task

#### Procedure

1.  Back up the following directories:

    -   `/etc/puppetlabs/puppet/ssl/`
    -   `/etc/puppetlabs/puppet/ssl/`
    -   `/etc/puppetlabs/puppetdb/ssl/`
    -   `/opt/puppetlabs/server/data/console-services/certs/`
    -   `/opt/puppetlabs/server/data/postgresql/9.6/data/certs/`
    -   `/etc/puppetlabs/orchestration-services/ssl`

### \(Optional\) Delete and recreate the master CA

If needed, you can delete and recreate the Puppet CA before regenerating the rest of your monolithic certificates.

#### About this task

|![](bolt-logo-dark.png)|As an alternative to performing these steps manually, on your master logged in as root, run `puppet infrastructure run rebuild_certificate_authority caserver=<CA_SERVER_HOSTNAME>`. If your master operates as your CA server, specify `caserver=localhost`. \(If you're running PE version 2018.1.11 or newer, omit the `caserver` parameter.\) Running the command with `localhost` avoids the requirement to set up SSH between your master and itself. The `puppet infrastructure run` command leverages built-in Bolt plans to automate certain management tasks. To use this command, you must be able to connect using SSH from your master to any nodes that the command modifies. You can establish an SSH connection using key forwarding, a local key file, or by specifying keys in `.ssh/config` on your master. For more information, see [Bolt OpenSSH configuration options](https://puppet.com/docs/bolt/latest/bolt_configuration_options.html#openssh-configuration-options).

To view all available parameters, use the `--help` flag. The logs for all `puppet infrastructure run` Bolt plans are located at `/var/log/puppetlabs/installer/bolt_info.log`.

|

CAUTION:

This is an optional step and is an meant for use in the event of a total compromise of your site, or some other unusual circumstance. This **destroys the certificate authority and all other certificates.**

Run the following commands on your master or CA server.

#### Procedure

1.  Delete the CA and clear all certs from your master: `rm -rf /etc/puppetlabs/puppet/ssl/*`

2.  Regenerate the CA: `puppet cert list -a`

    You should see this message: `Notice: Signed certificate request for ca`


### Regenerate the Puppet master certificates

In this step, you'll create the certificates for the Puppet master and then configure PE so the certificate is available to PE's components and services.

#### About this task

|![](bolt-logo-dark.png)|As an alternative to performing these steps manually, on your master logged in as root, run `puppet infrastructure run regenerate_master_certificate`.

 You can specify these optional parameters:

-   `tmpdir` — Path to a directory to use for uploading and executing temporary files.
-   `dns_alt_names` – Comma-separated list of alternate DNS names to be added to the certificates generated for your master.

**Important:** To use the `dns_alt_names` parameter, you must configure Puppet Server with `allow-subject-alt-names` in the `certificate-authority` section of `pe-puppet-server.conf`. To ensure naming consistency, if your `puppet.conf` file includes a `dns_alt_names` entry, you must include the `dns_alt_names` parameter and pass in all alt names included in the entry when regenerating certificates.


 The `puppet infrastructure run` command leverages built-in Bolt plans to automate certain management tasks. To use this command, you must be able to connect using SSH from your master to any nodes that the command modifies. You can establish an SSH connection using key forwarding, a local key file, or by specifying keys in `.ssh/config` on your master. For more information, see [Bolt OpenSSH configuration options](https://puppet.com/docs/bolt/latest/bolt_configuration_options.html#openssh-configuration-options).

 To view all available parameters, use the `--help` flag. The logs for all `puppet infrastructure run` Bolt plans are located at `/var/log/puppetlabs/installer/bolt_info.log`.

|

#### Procedure

1.  Remove the Puppet master's cached catalog:`rm -f /opt/puppetlabs/puppet/cache/client_data/catalog/<CERTNAME>.json`

2.  Clear the cert for the Puppet master:`puppet cert clean <CERTNAME>`

    **Note:** This step is not necessary if you deleted and recreated the CA cert.

3.  Generate the certificates for PE services and update the configuration of PE: `puppet infrastructure configure --no-recover`

    **Note:** Be sure to specify any DNS alt names you have in the `pe_install::puppet_master_dnsaltnames` array in `/etc/puppetlabs/enterprise/conf.d/pe.conf`. You can find the list of your current DNS alt names with `puppet cert list <CERTNAME>`. By default, PE uses `puppet` and `puppet.domain`.

4.  Run Puppet on the Puppet master: `puppet agent -t`

    A successful Puppet run is necessary to ensure that PE's services are properly configured.lan


**Related information**  


[Regenerate compile master certs](regenerate_compile_master_certificates.md)

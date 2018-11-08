---
layout: default
title: " PE 3.8 » Razor » Install Razor"
subtitle: "Install and Set Up Razor"
canonical: "/pe/latest/razor_install.html"

---
A Razor module is included with Puppet Enterprise. To install and configure a Razor server, you must [set up your Razor environment](./razor_prereqs.html), and then classify the `pe_razor` node. When PE runs and applies this Razor classification, the Razor server and a PostgreSQL database will be installed and configured.

In addition to the Razor server, the Razor client can be installed as a Ruby gem on any machine you want to use for interacting with Razor. The client makes interacting with the server from the command line easier. It lets you explore what the server knows about your infrastructure; for example, it helps avoid datatype errors, and enables you to modify how machines are provisioned by interacting with the Razor server API.

If you're not a PE user, you can install the [open source version of Razor manually](https://github.com/puppetlabs/razor-server/wiki/Installation).

**Important**: We highly recommend that you set Razor up in a completely isolated test environment before you run it on your production environment. This environment must have access to the internet. See [Install and Set Up an Environment for Razor](./razor_prereqs.html) for details.

### Before You Begin
Things you should know before you set up provisioning:

+ **Do not** install Razor on your Puppet master.
+ The default ports for Razor are port 8150 for HTTP communication between the server and nodes, and port 8151 for HTTPS, used for accessing the public API. You can change the default, as described in the "Changing the Default Razor Port" section below.
+ Running a Razor server is supported on RHEL/CentOS 6.x and 7.x.

>**Hint**: With the `export` command, you can avoid having to repeatedly replace placeholder text. The steps for installing assume you have declared a server name and the port to use for Razor with this command:
>
>     export RAZOR_HOSTNAME=<server name>
>     export HTTP_PORT=8150
>     export HTTPS_PORT=8151
>
> The steps below therefore use `$RAZOR_HOSTNAME`, `$HTTP_PORT` and `$HTTPS_PORT` for brevity.

Install the Razor Server
-------------

The actual Razor software is stored in an external online location, so you need an internet connection to install it. The process entails classifying a node with the `pe_razor` module. When you do so, the software is downloaded. This process can take several minutes.

Manually add the `pe_razor` class in the PE console, as follows.

1. In the console, click **Classification**, and then click the node group you will use to assign the `pe-razor` class to nodes.
2. Click **Classes** and, in **Class name**, type "pe_razor".
3. Click **Add class** and click the commit button. For information about adding a  class and classifying the Razor server using the PE console, see the [Adding Nodes to a Node Group](./console_classes_groups.html#adding-nodes-to-a-node-group) and [Adding New Classes](./console_classes_groups.html#adding-classes-to-a-node-group) sections of this guide.


	**Note**: You can also add the following to site.pp (provide your own agent cert info):

		node <AGENT_CERT>{
			include pe_razor
		}

4. On the Razor server, run Puppet with: `puppet agent -t` (otherwise you have to wait for the scheduled agent run).

#### Installing the Razor Server While You're Offline

If you don't have access to the internet or would like to pull the PE tarball from your own location, you can do as follows.

1. In the console, in the **pe_razor** class that you added in the previous section, click the **Parameter name** down arrow, and select **pe_tarball_base_url**.
2. In the **Value** box, type in your own URL, and click **Add parameter**.

	The tarball must retain the same name format as on our server. The default URL is `https://pm.puppetlabs.com/puppet-enterprise/${pe_build}/puppet-enterprise-${pe_build}-${relarch}.tar.gz`. For example, `https://pm.puppetlabs.com/puppet-enterprise/3.8.0/puppet-enterprise-3.8.0-el-6-x86_64.tar.gz`.

	Substitute your own hosting for this part of the URL, `https://pm.puppetlabs.com/puppet-enterprise`. The latter part needs to contain the information for the tarball you're using in this format:  `puppet-enterprise-${pe_build}-${relarch}.tar.gz`.

	See a list of [the versions of the tarball](install_basic.html#choosing-an-installer-tarball) for more information. Available tarballs are located on the [Puppet Enterprise downloads page](https://puppetlabs.com/misc/pe-files).

3. Also in the **pe_razor** class area, add the `microkernel_url` parameter, and in the **Value** box, add the URL for the microkernel. The URL might be to your own FTP site. Or, you can copy the microkernel onto the Razor server and then use, `file:///path/to/microkernel.tar` for the URL.

	The microkernel is called [razor-microkernel-latest.tar](https://pm.puppetlabs.com/puppet-enterprise-razor-microkernel-3.8.0.tar).

4. Commit your changes.


#### The `pe_razor` Module

The `pe_razor` module has the following parameters:

| Parameter | Description |
|-----------| ------------|
| `dbpassword` | The database password to use for Razor's database. The password defaults to `razor`. |
| `pe_tarball_base_url` | The location of the Puppet Enterprise tarball. |
| `microkernel_url` | The URL from which to fetch the microkernel. |
| `server_http_port` | The port that nodes use to communicate with the server over HTTP. Only URLs starting with /svc need to be available on this port. It defaults to 8150. |
| `server_https_port` | The port that the client uses to communicate with the server's public API over HTTPS. Only URLs starting with /api need to be available on this port. It defaults to 8151. |


#### Changing the Default Razor Port

If you want to change the default HTTP or HTTPS port, you can make the change in the console, like this:

1. On the **Classification** tab, click the node group that contains the `pe_razor` module.
2. Click the **Classes** tab. Then, under **pe_razor** in the **Parameters** box, select the parameter you want to change, such as `server_http_port` or `server_https_port`.
3. In the **Value** box, type the port number you want to use, then click **Add parameter** and the commit change button.


### Load iPXE Software

You must set your machines to PXE boot. If they don't PXE boot, Razor has no way to interact with a system. When a node has already been enrolled with Razor and is installed, it no longer has to PXE boot, but that will prevent any changes on the server (for example, an attempt to reinstall the system) from having any effect on the node. Razor relies on "seeing" when a machine boots and starts all its interactions with a node when that node boots.

Razor provides a specific iPXE boot image to ensure you're using a compatible version.

1. Download the iPXE boot image [undionly-20140116.kpxe](http://links.puppetlabs.com/pe-razor-ipxe-firmare-3.3).
2. Copy the image to `/var/lib/tftpboot`: `cp undionly-20140116.kpxe /var/lib/tftpboot`.

3. Download the iPXE bootstrap script from the Razor server to the `/var/lib/tftpboot` directory:

	`wget 	"https://${RAZOR_HOSTNAME}:${HTTPS_PORT}/api/microkernel/bootstrap?nic_max=1&http_port=${HTTP_PORT}" -O /var/lib/tftpboot/bootstrap.ipxe`

 **Note**: Make sure you don't use `localhost` as the name of the Razor host. The bootstrap script chain-loads the next iPXE script from the Razor server. This means that it has to contain the correct hostname for clients to try and fetch that script from, or it isn't going to work.


### Verify the Razor Server

Test the new Razor configuration: `wget https://${RAZOR_HOSTNAME}:${HTTPS_PORT}/api -O test.out`.

The command should execute successfully, and the output JSON file `test.out` should contain a list of available Razor commands.


### Install and Set Up the Razor Client

The Razor client is installed as a Ruby gem.

1. Install the client:

		gem install pe-razor-client --version 1.0.0

2. Point the Razor client to the server:

        razor -u https://${RAZOR_HOSTNAME}:${HTTPS_PORT}/api.

   An error displays if the client isn't installed or can't connect to the server.

3. You'll likely get this warning message about JSON: "MultiJson is using the default adapter (ok_json). We recommend loading a different JSON library to improve performance."  This message is harmless, but you can disble it with this command:

		gem install json_pure

**Note**: There is also a `razor-client` gem that contains the client for the open source Razor client. We strongly recommended that you not install the two clients simultaneously, and that you only use `pe-razor-client` with the Razor shipped as part of Puppet Enterprise. If you already have `razor-client` installed, or are not sure if you do, run `gem uninstall razor-client` prior to step (1) above.

### Verify Razor versions

Verify your installation by checking the version of the Razor server and client. This information can also be useful for troubleshooting.

1. On the client or server, enter `razor --version` or `razor -v`.

### Uninstall Razor

To uninstall the Razor Server:

1. Run `yum erase pe-razor`.
2. Drop the PostgreSQL database that the server used.
3. Change DHCP/TFTP so that the machines that have been installed will continue to boot outside the scope of Razor.

To uninstall the Razor client:

+  Run `gem uninstall pe-razor-client`.



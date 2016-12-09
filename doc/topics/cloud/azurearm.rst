==============================
Getting Started With Azure ARM
==============================

.. versionadded:: 2016.11.0

Azure is a cloud service by Microsoft providing virtual machines, SQL services,
media services, and more. Azure ARM (aka, the Azure Resource Manager) is a next
generatiom version of the Azure portal and API. This document describes how to
use Salt Cloud to create a virtual machine on Azure ARM, with Salt installed.

More information about Azure is located at `http://www.windowsazure.com/
<http://www.windowsazure.com/>`_.


Dependencies
============
* `Microsoft Azure SDK for Python <https://pypi.python.org/pypi/azure>`_ >= 2.0rc5
* `Microsoft Azure Storage SDK for Python <https://pypi.python.org/pypi/azure-storage>`_ >= 0.32
* The python-requests library, for Python < 2.7.9.
* A Microsoft Azure account
* `Salt <https://github.com/saltstack/salt>`_


Configuration
=============

Set up the provider config at ``/etc/salt/cloud.providers.d/azurearm.conf``:

.. code-block:: yaml

    # Note: This example is for /etc/salt/cloud.providers.d/azurearm.conf

    my-azurearm-config:
      driver: azurearm
      master: limejack.com
      subscription_id: 01234567-890a-bcde-f012-34567890abdc

      # https://apps.dev.microsoft.com/#/appList
      username: <username>@<subdomain>.onmicrosoft.com
      password: verybadpass
      location: westus
      resource_group: my_rg

      # Optional
      network_resource_group: my_net_rg
      cleanup_disks: True
      cleanup_vhds: True
      cleanup_data_disks: True
      cleanup_interfaces: True
      custom_data: 'This is a Joseph test'
      expire_publisher_cache: 604800  # 1 week
      expire_offer_cache: 604800  # 1 week
      expire_sku_cache: 604800  # 1 week
      expire_version_cache: 604800  # 1 week
      expire_group_cache: 86400  # 1 day
      expire_interface_cache: 3600  # 1 hour
      expire_network_cache: 3600  # 1 hour
      expire_subnet_cache: 3600  # 1 hour

Cloud Profiles
==============
Set up an initial profile at ``/etc/salt/cloud.profiles``:

.. code-block:: yaml

    azure-ubuntu:
      provider: my-azure-config
      image: Canonical|UbuntuServer|14.04.5-LTS|14.04.201612050
      size: Standard_D1_v2
      location: eastus
      ssh_username: azureuser
      ssh_password: verybadpass

    azure-win2012:
      provider: my-azure-config
      image: MicrosoftWindowsServer|WindowsServer|2012-R2-Datacenter|latest
      size: Standard_D1_v2
      location: westus
      win_username: azureuser
      win_password: verybadpass

These options are described in more detail below. Once configured, the profile
can be realized with a salt command:

.. code-block:: bash

    salt-cloud -p azure-ubuntu newinstance

This will create an salt minion instance named ``newinstance`` in Azure. If
the command was executed on the salt-master, its Salt key will automatically
be signed on the master.

Once the instance has been created with salt-minion installed, connectivity to
it can be verified with Salt:

.. code-block:: bash

    salt newinstance test.ping


Profile Options
===============
The following options are currently available for Azure ARM.

provider
--------
The name of the provider as configured in
`/etc/salt/cloud.providers.d/azure.conf`.

image
-----
Required. The name of the image to use to create a VM. Available images can be
viewed using the following command:

.. code-block:: bash

    salt-cloud --list-images my-azure-config

As you will see in ``--list-images``, image names are comprised of the following
fields, separated by the pipe (``|``) character:

.. code-block:: yaml

    publisher: For example, Canonical or MicrosoftWindowsServer
    offer: For example, UbuntuServer or WindowsServer
    sku: Such as 14.04.5-LTS or 2012-R2-Datacenter
    version: Such as 14.04.201612050 or latest

It is possible to specify the URL of a custom image that you have access to,
such as:

.. code-block:: yaml

    https://<mystorage>.blob.core.windows.net/system/Microsoft.Compute/Images/<mystorage>/template-osDisk.01234567-890a-bcdef0123-4567890abcde.vhd

size
----
Required. The name of the size to use to create a VM. Available sizes can be
viewed using the following command:

.. code-block:: bash

    salt-cloud --list-sizes my-azure-config

location
--------
Required. The name of the location to create a VM in. Available locations can
be viewed using the following command:

.. code-block:: bash

    salt-cloud --list-locations my-azure-config

ssh_username
------------
Required for Linux. The user to use to log into the newly-created Linux VM to
install Salt.

ssh_password
------------
Required for Linux. The password to use to log into the newly-created Linux VM
to install Salt.

win_username
------------
Required for Windows. The user to use to log into the newly-created Windows VM
to install Salt.

win_password
------------
Required for Windows. The password to use to log into the newly-created Windows
VM to install Salt.

resource_group
--------------
Required. The resource group that all VM resources (VM, network interfaces,
etc) will be created in.

network_resource_group
----------------------
Optional. If specified, then the VM will be connected to the network resources
in this group, rather than the group that it was created in. The VM interfaces
and IPs will remain in the configured ``resource_group`` with the VM.

cleanup_disks
-------------
Optional. Default is ``False``. If set to ``True``, disks will be cleaned up
when the VM that they belong to is deleted.

cleanup_vhds
------------
Optional. Default is ``False``. If set to ``True``, VHDs will be cleaned up
when the VM and disk that they belong to are deleted. Requires ``cleanup_disks``
to be set to ``True``.

cleanup_data_disks
------------------
Optional. Default is ``False``. If set to ``True``, data disks (non-root
volumes) will be cleaned up whtn the VM that they are attached to is deleted.
Requires ``cleanup_disks`` to be set to ``True``.

cleanup_interfaces
------------------
Optional. Default is ``False``. Normally when a VM is deleted, its associated
interfaces and IPs are retained. This is useful if you expect the deleted VM
to be recreated with the same name and network settings. If you would like
interfaces and IPs to be deleted when their associated VM is deleted, set this
to ``True``. 

custom_data
-----------
Any custom cloud data that needs to be specified. How this data is used depends
on the operating system and image that is used. For instance, Linux images that
use ``cloud-init`` will import this data for use with that program. Some
Windows images will create a file with a copy of this data, and others will
ignore it. If a Windows image creates a file, then the location will depend
upon the version of Windows.

expire_publisher_cache
----------------------
Optional. Default is ``604800``. When fetching image data using
``--list-images``, a number of web calls need to be made to the Azure ARM API.
This is normally very fast when performed using a VM that exists inside Azure
itself, but can be very slow when made from an external connection.

By default, the publisher data will be cached, and only updated every ``604800``
seconds (7 days). If you need the publisher cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the publisher
cache.

expire_offer_cache
------------------
Optional. Default is ``604800``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the offer data will be cached, and only updated every ``604800``
seconds (7 days). If you need the offer cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the publiser
cache.

expire_sku_cache
----------------
Optional. Default is ``604800``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the sku data will be cached, and only updated every ``604800``
seconds (7 days). If you need the sku cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the sku
cache.

expire_version_cache
--------------------
Optional. Default is ``604800``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the version data will be cached, and only updated every ``604800``
seconds (7 days). If you need the version cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the version
cache.

expire_group_cache
------------------
Optional. Default is ``604800``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the resource group data will be cached, and only updated every
``604800`` seconds (7 days). If you need the resource group cache to be updated
at a different frequency, change this setting. Setting it to ``0`` will turn
off the resource group cache.

expire_interface_cache
----------------------
Optional. Default is ``3600``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the interface data will be cached, and only updated every ``3600``
seconds (1 hour). If you need the interface cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the interface
cache.

expire_network_cache
--------------------
Optional. Default is ``3600``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the network data will be cached, and only updated every ``3600``
seconds (1 hour). If you need the network cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the network
cache.

expire_subnet_cache
-------------------
Optional. Default is ``3600``. See ``expire_publisher_cache`` for details on
why this exists.

By default, the subnet data will be cached, and only updated every ``3600``
seconds (1 hour). If you need the subnet cache to be updated at a different
frequency, change this setting. Setting it to ``0`` will turn off the subnet
cache.

Show Instance
=============
This action is a thin wrapper around ``--full-query``, which displays details on
a single instance only. In an environment with several machines, this will save
a user from having to sort through all instance data, just to examine a single
instance.

.. code-block:: bash

    salt-cloud -a show_instance myinstance
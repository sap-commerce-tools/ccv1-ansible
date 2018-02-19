# Ansible Playbooks for hybris cloud services

Here you can find ansbile playbooks to install a hcs deployment package to open environments

It only supports stop-the-world deployments for now, with or without running hybris update.

Following optimizations are present:

- environment specific configuration is loaded via the [optional configuration directory](https://help.hybris.com/6.6.0/hcd/8beb75da86691014a0229cf991cb67e4.html)
- deployment does **not** perform a full build


## Prerequisites

- hcs VPN connection
- hcs ssh access\
  (use the file  `group_vars/all` to define the default login user or `~/.ssh/config`)
- hcs user has sudo rights (can change to `root` and `hybris`)
- deployment package (zip and md5) is available in `/NFS_DATA/deployment`

## Tips and Tricks

- keep a separate [inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html) file for every hcs environment, to minimize errors
- please use the same inventory structure as outlined in the file `inventory_example`

## Usage

## Step 1: Install new release on every hybris server

```
ansible-playbook -i <inventory> install-release.yaml --extra-vars "package=<package name>"
```

This command sets up a new version on every hybris server (target folder: `/opt/{{package}}`),
with configurations, symlinks etc. as [recommended by hcs](https://help.hybris.com/6.6.0/hcd/9857451294254ae1a5ffcdcc8fd63b7d.html).

The package is [validated](https://help.hybris.com/6.6.0/hcd/89922e5cdfeb41d99725f2901233296e.html) before installation, as is the available disk sapce
(Default: at least 2.5 GiB must be free, see variable `minimal_available_space`)

|Parameter|Description|
|---|---|
|`<inventory>`|Which inventory to use, in other words: which environment to deploy to|
|`package=<package name>`|Name of the package (filename without the `.zip` extension)|


### Example

```
ansible-playbook -i dev install-release.yaml --extra-vars "package=acme-ts_v1.0.0"
```

## Step 2: Activate (new) release

```
ansible-playbook -i <inventory> activate-release.yaml --extra-vars "package=<package name> update=True|False"
```

Configures the specified version (available in folder `/opt/{{package}}`) as the new
active installation on all hybris servers, and optionally performs a hybris update running system

|Parameter|Description|
|---|---|
|`<inventory>`|Which inventory to use, in other words: which environment to deploy to|
|`package=<package name>`|Name of the package (filename without the `.zip` extension)|
|`update=True\|False`|Run hybris update?|

### Example

```
ansible-playbook -i qa activate-release.yaml --extra-vars "package=acme-ts_v1.0.0 update=true"
```

### Notes

If the file `/opt/{{package}}/data/misc/update.json` is available, the hybris update will
use this file as configuration (using `-DconfigFile=...`).
Otherwise, a default hybris update is executed.

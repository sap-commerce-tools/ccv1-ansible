# Ansible Playbooks for Commerce Cloud **v1**
[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/W7W7VS24)

Here you can find Ansbile playbooks to install a hcs deployment package to open environments

It only supports stop-the-world deployments for now, with or without running hybris update.

Following optimizations are present:

- environment specific configuration is loaded via the [optional configuration directory](https://help.hybris.com/6.6.0/hcd/8beb75da86691014a0229cf991cb67e4.html)
- deployment does **not** perform a full build


## Prerequisites

- hcs VPN connection
- hcs ssh access\
  (use the file  `group_vars/all` to define the default login user, or setup `~/.ssh/config`)
- hcs user has sudo rights (can change to `root` and `hybris`)

## Tips and Tricks

- keep a separate [inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html) file for every hcs environment, to minimize the possibility for errors
- please use the same inventory structure as outlined in the file `inventory_example`

## Usage

## Step 1: Install new release on every hybris server

```
ansible-playbook -i <inventory> install-release.yaml --extra-vars "package=<package name>"
```

This command sets up a new version on every hybris server (target folder: `/opt/{{package}}`),
with configurations, symlinks etc. as [recommended by hcs][dev-install]

The package is [validated][validation] before installation, as is the available disk sapce
(Default: at least 2.5 GiB must be free, see variable `minimal_available_space` defined in `group_vars/hybris`)

The playbook assumes that the package (zip file and md5 checksum file, as defined in the 
[package structure][package]) is already available in your hcs environment 
(uploaded to `/NFS_DATA/deployment/<package name>.{md5,zip}`)

If you want to automate the upload too, define the path where the new package files (zip, md5)
are ready to upload using the variable `hcs_artifact_folder` \
(see file `group_vars/hybris` for a commented example).

|Parameter|Description|
|---|---|
|`<inventory>`|Which inventory to use, in other words, which environment to deploy to|
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

Since activating a new release is separate from installing, you can easily roll
back to a previous version in case something goes wrong.

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
use this file as configuration (using `ant updatesystem -DconfigFile=...`).
Otherwise, a default hybris update (`ant updatesystem` without any parameters) is executed.

## Delete old releases

```
ansible-playbook -i <inventory> delete-release.yaml --extra-vars "package=<package name>"
```

Deletes files in `/NFS_DATA/deployment` and `/opt` (on every node) related to a 
package.

|Parameter|Description|
|---|---|
|`<inventory>`|Which inventory to use, in other words: which environment to deploy to|
|`package=<package name>`|Name of the package (filename without the `.zip` extension)|

### Example

```
ansible-playbook -i qa delete-release.yaml --extra-vars "package=acme-ts_v1.0.0"
```


[package]: https://help.hybris.com/scc/sid/b2d62e2a93004c6c9c3d438c4dc31fba.html
[validation]: https://help.hybris.com/scc/sid/89922e5cdfeb41d99725f2901233296e.html
[dev-install]: https://help.hybris.com/scc/sid/4906e3a1b3a340faaaafb64639bf191d.html

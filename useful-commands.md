# Ansible Snippets

Some useful commands for open hcs environments

## Start/Stop hybris on all nodes

Use `state=started` to launch the service, or `state=stopped` to stop it.
The `--become` flag tells ansible to use `sudo` to perform the action

    ansible -i <inventory> hybris -m service -a "name=hybris state=stopped" --become
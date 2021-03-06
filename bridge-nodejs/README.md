### Prerequisites
1. A functional Ubuntu 16.04 server launched using a trusted hosting provider. For more information, see our tutorials on [setting up a validator node on AWS](https://github.com/poanetwork/wiki/wiki/Validator-Node-on-AWS) or [setting up on non-AWS](https://github.com/poanetwork/wiki/wiki/Validator-Node-Non-AWS).
   * Record the IP address (required for file setup).
   * Setup ssh access to your node via public+private keys (using passwords is less secure). 
   * When creating the node, set a meaningful `hostname` that can identify you (e.g. `validator-0x...`).

2. On your local machine install:
    * Python 2 (v2.6-v2.7)/Python3 (v3.5+)
    * Ansible v2.3+
    * Git

### Configuration

1. Clone this repository and go to the `bridge-nodejs` folder
```
git clone https://github.com/poanetwork/deployment-bridge.git
cd bridge-nodejs
```
2. Create the file `hosts.yml` from `hosts.yml.example`
```
cp hosts.yml.template hosts.yml
```

`hosts.yml` should have the following structure:

```yaml
<bridge_name>:
    hosts:
        <host_ip>:
            ansible_user: <user>
            VALIDATOR_ADDRESS_PRIVATE_KEY: "<private_key>"
            #syslog_server_port: "<protocol>://<ip>:<port>" # When this parameter is set all bridge logs will be redirected to <ip>:<port> address.
```

| Value | Description |
|:------------------------------------------------|:----------------------------------------------------------------------------------------------------------|
| `<bridge_name>` | The bridge name which tells Ansible which file to use. This is located in `group_vars/<bridge_name>.yml`. |
| `<host_ip>` | Remote server IP address. |
| ansible_user: `<user>` | User that will ssh into the node. This is typically `ubuntu` or `root`. |
| VALIDATOR_ADDRESS_PRIVATE_KEY: `"<private_key>"` | The private key for the specified validator address. |
| syslog_server_port: `"<protocol>://<ip>:<port>"` | Optional port specification for bridge logs. This value will be provided by an administrator if required.  |


`hosts.yml` can contain multiple hosts and bridge configurations (groups) at once.


3. Copy the bridge name(s) to the hosts.yml file. 
   1. Go to the group_vars folder. 
   `cd group_vars`
   2. Note the  <bridge_name> and add it to the hosts.yml configuration. For example, if a bridge file is named sokol-kovan.yml, you would change the <bridge_name> value in hosts.yml to sokol-kovan.

#### Administrator Configurations

1. The `group_vars/<bridge_name>.yml` file contains the public bridge parameters. This file is prepared by administrators for each bridge. The validator only needs to add the required bridge name in the hosts.yml file to tell Ansible which file to use.

   `group_vars/example.yml` shows an example configuration for the POA/Sokol - POA/Sokol bridge. Parameter values should match values from the .env file for the token-bridge. See https://github.com/poanetwork/token-bridge#configuration-parameters for details.

2. Set the `versions` parameter in the `roles/repo/tasks/main.yml` file to define a specific branch or commit to use with the [token bridge](https://github.com/poanetwork/token-bridge). If `versions` is not specified, the default (`master`) branch is used.

   In this example, the `support-erc20-native-#81` branch is used.

```
- name: Get bridge repo
  git:
    repo: "{{ bridge_repo }}"
    dest: "{{ bridge_path }}"
    force: yes
    version: support-erc20-native-#81
```

## Execution

The playbook can be executed once [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) is installed and all configuration variables are complete. 

It will automatically install `Docker`, `docker-compose`, `Python`, `Git` and it dependencies (such as `curl`, `ca-certificates`, `apt-transport-https`, etc.) to the node. 

```yaml
ansible-playbook -i hosts.yml site.yml
```

**Useful arguments:**

To be used with the ansible-playbook command, for example:

```yaml
 `ansible-playbook -i hosts.yml site.yml --ask-become-pass`
```

* `--ask-pass` - ask for the password used to connect to the bridge VM.

* `--ask-become-pass` - ask for the `become` password used to execute some commands (such as Docker installation) with root privileges.

* `-i <file>` - use specified file as a `hosts.yml` file.

* `-e "<variable>=<value>"` - override default variable.

* `--private-key=<file_name>` - if private keyfile is required to connect to the ubuntu instance.

**Useful variables:**

These variables can be added to hosts.yml to override defaults.

* `bridge_path` - absolute path where bridge will be installed. Defaults to /home/<ansible_user>/bridge.

* `bridge_repo` - path to the bridge repo.

* `docker_compose_version` - specify a version of docker-compose to be installed.


## Bridge service commands

The Bridge service is named `poabridge`. Use the default `SysVinit` commands to `start`, `stop`, `restart`, and `rebuild` the service and to check the `status` of the service. 

Commands format:
```bash
sudo service poabridge [start|stop|restart|status|rebuild]
```

## Logs

If the `syslog_server_port` option in the hosts.yml file was not set, all logs will be stored in containers and can be accessed via the default `docker-compose logs` command (use `sudo` if necessary). 

To obtain logs, `cd` to the directory where the bridge is installed (`~/bridge` by default) and execute the `sudo docker-compose logs` command.

If the `syslog_server_port` was set, logs can be obtained from the specified server.

```yaml 
syslog_server_port: "<protocol>://<ip>:<port>" # When this parameter is set all bridge logs will be redirected to the <ip>:<port> address.
```

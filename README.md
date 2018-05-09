sFlow role
==========

This role facilitates the configuration of global and interface level sFlow attributes. It supports the configuration of sFlow collectors at the global level, enable/disable, and specification of sFlow polling-interval, sample-rate, max-datagram size, and so on are supported at the interface and global level. This role is abstracted for dellos9.

The dellos-sflow role requires an SSH connection for connectivity to a Dell EMC Networking device. You can use any of the built-in OS connection variables.

Installation
------------

    ansible-galaxy install Dell-Networking.dellos-sflow

Role variables
--------------

- Role is abstracted using the *ansible_network_os* variable that can take the dellos9 value
- If *dellos_cfg_generate* is set to true, the variable generates the role configuration commands in a file
- Any role variable with a corresponding state variable set to absent negates the configuration of that variable
- Setting an empty value for any variable negates the corresponding configuration
- *dellos_sflow* (dictionary) contains keys along with *interface name* (dictionary)
- Interface name can correspond to any of the valid dellos9 physical interfaces with the unique interface identifier name
- Interface name must be in *<interfacename> <tuple>* format; physical interface name can be in *fortyGigE 1/1* format for dellos9 devices
- Variables and values are case-sensitive

**dellos_sflow keys**

| Key        | Type                      | Description                                             | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``sflow_enable`` | boolean: true,false\* | Enables sFlow at a global level  | dellos9 |
| ``collector``    | list                  | Configures collector information (see ``collector.*``); only two collectors can be configured on dellos9 devices | dellos9 |
| ``collector.collector_ip`` | string (required) | Configures an IPv4/IPv6 address for the collector | dellos9 |
| ``collector.agent_addr`` | string (required) | Configures an IPv4/IPv6 address for the sFlow agent to the collector | dellos9 |
| ``collector.udp_port`` | integer | Configures UDP port range at the collector level (1 to 65535) | dellos9 |
| ``collector.max_datagram_size`` | integer | Configures the maximum datagram size for the sFlow datagrams generated (400 to 1500) | dellos9 |
| ``collector.vrf`` | boolean: true,false* | Configures the management VRF to reach collector if set to true; can be enabled only for IPv4 collector addresses | dellos9 |
| ``polling_interval`` | integer | Configures the global default counter polling-interval (15 to 86400) | dellos9 |
| ``sample_rate`` | integer | Configures the global default sample-rate (256 to 8388608) | dellos9 |
| ``extended_switch`` | boolean: true,false\* | Enables packing extended information for the switch if set to true | dellos9  |
| ``max_header_size`` | boolean: true,false\* | Enables extended header copy size of 256 bytes if set to true at the global level | dellos9 |

> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

**interface name keys**

| Key        | Type                      | Notes                                                   |
|------------|---------------------------|---------------------------------------------------------|
| ``sflow_enable`` | boolean: true,false\*   | Enables sFlow at the interface level  | 
| ``ingress_enable`` | boolean: true,false\* | Enables ingress sFlow at the interface level  | 
| ``polling_interval`` | integer | Configures the interface level default counter polling-interval (15 to 86400) |
| ``max_header_size`` | boolean: true,false\* | Enables extended header copy size of 256 bytes if set to true at the interface level |  
| ``sample_rate`` | integer | Configures the interface level default sample-rate (256 to 8388608) | 

> **NOTE**: Asterisk (\*) denotes the default value if none is specified. 

Connection variables
--------------------

Ansible Dell EMC Networking roles require connection information to establish communication with the nodes in your inventory. This information can exist in the Ansible *group_vars* or *host_vars* directories,or inventory or in the playbook itself.

| Key         | Required | Choices    | Description                                         |
|-------------|----------|------------|-----------------------------------------------------|
| ``ansible_host`` | yes      |            | Specifies the hostname or address for connecting to the remote device over the specified transport |
| ``ansible_port`` | no       |            | Specifies the port used to build the connection to the remote device; if value is unspecified, the ANSIBLE_REMOTE_PORT option is used; it defaults to 22 |
| ``ansible_ssh_user`` | no       |            | Specifies the username that authenticates the CLI login for the connection to the remote device; if value is unspecified, the ANSIBLE_REMOTE_USER environment variable value is used  |
| ``ansible_ssh_pass`` | no       |            | Specifies the password that authenticates the connection to the remote device |
| ``ansible_become`` | no       | yes, no\*   | Instructs the module to enter privileged mode on the remote device before sending any commands; if value is unspecified, the ANSIBLE_BECOME environment variable value is used, and the device attempts to execute all commands in non-privileged mode |
| ``ansible_become_method`` | no       | enable, sudo\*   | Instructs the module to allow the become method to be specified for handling privilege escalation; if value is unspecified, the ANSIBLE_BECOME_METHOD environment variable value is used |
| ``ansible_become_pass`` | no       |            | Specifies the password to use if required to enter privileged mode on the remote device; if ``ansible_become`` is set to no this key is not applicable |
| ``ansible_network_os`` | yes      | dellos6/dellos9/dellos10, null\*  | Loads the correct terminal and cliconf plugins to communicate with the remote device |

> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

Dependencies
------------

The *dellos-sflow* role is built on modules included in the core Ansible code. These modules were added in Ansible version 2.2.0.

Example playbook
----------------

This example uses the *dellos.dellos-sflow* role to configure sFlow attributes at interface and global level. It creates a *hosts* file with the switch details and corresponding variables. The hosts file should define the *ansible_network_os* variable with the corresponding Dell EMC networking OS name. 

When *dellos_cfg_generate* is set to true, the variable generates the configuration commands as a .part file in *build_dir* path. By default, the variable is set to false. It writes a simple playbook that only references the *dellos-sflow* role. By including the role, you automatically get access to all of the tasks to configure sFlow features. 

**Sample hosts file**
 
    leaf1 ansible_host= <ip_address> 

**Sample host_vars/leaf1**

    hostname: leaf1
    ansible_become: yes
    ansible_become_method: xxxxx
    ansible_become_pass: xxxxx
    ansible_ssh_user: xxxxx
    ansible_ssh_pass: xxxxx
    ansible_network_os: dellos9
    build_dir: ../temp/dellos9
    dellos_sflow:
      sflow_enable: true
      collector:
        - collector_ip: 1.1.1.1
          agent_addr: 2.2.2.2
          udp_port: 2
          max_datagram_size: 1000
          vrf: true
          state: present
      polling_interval: 30
      sample_rate: 1024
      extended_switch : true
      max_header_size: true
      fortyGigE 1/1:
        sflow_enable : true
        ingress_enable: true
        polling_interval: 30
        sample_rate: 1024
        max_header_size: true

**Simple playbook to setup sflow - leaf.yaml**

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-sflow

**Run**

    ansible-playbook -i hosts leaf.yaml

(c) 2017 Dell Inc. or its subsidiaries. All Rights Reserved.

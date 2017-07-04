Sflow Role for Dell EMC Networking OS
======================================

This role facilitates the configuration of global and interface level sflow attributes. It supports the configuration of sflow collectors at global level. Enabling and disabling of sflow and specification of sflow polling-interval, sample-rate, max-datagram size , etc; are supported at interface and global level.This role is abstracted for dellos9.


Installation
------------

```
ansible-galaxy install Dell-Networking.dellos-sflow
```

Requirements
------------
This role requires an SSH connection for connectivity to your Dell EMC Networking device. You can use any of the built-in Dell EMC Networking OS connection variables, or the ``provider``
dictionary.

Role Variables
--------------

Any role variable with a corresponding state variable set to absent negates the configuration of that variable. For variables with no state variable, setting an empty value for the variable negates the corresponding configuration.

The variables and its values are case-sensitive.

``dellos_sflow`` dictionary contains the following keys along with ``interface name `` dictionary.
The interface name can correspond to any of the valid dellos9 physical interfaces with the unique interface identifier name.
The interface name must be in the format `<interfacename> <tuple>`. 
For example, the physical interface name can be in the format fortyGigE 1/1 for dellos9 devices.

|        Key | Type                      | Notes                                                                                                                                                                                     |
|------------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| sflow_enable | boolean: true, false* | Enables sflow at the global level.  |
| collector    | list                  | Configures collector information. See the following collector.* keys for each list item. Only two collectors can be configured on dellos9 devices. |
| collector.collector_ip | string (required) | Configures ipv4/ipv6 address for the collector. |
| collector.agent_addr | string (required) | Configures ipv4/ipv6 address for the sflow agent to the collector. |
| collector.udp_port | integer | Configures udp-port at the collector level. It can be specified in the range 1-65535. |
| collector.max_datagram_size | integer | Configures maximum datagram size for the sflow datagrams generated. It can be in the range 400-1500. |
| collector.vrf | boolean: true, false* | Configures management vrf to reach collector if this key is set to true. It can be enabled only for ipv4 collector addresses. |
| polling_interval | integer | Configures global default counter polling-interval. It can be specified in the range 15-86400.|
| sample_rate | integer | Configures global default sample-rate. It can be specified in the range 256-8388608. |
| extended_switch | boolean: true,false* | Enable packing extended information for the switch if the key is set to true. |
| max_header_size | boolean: true,false* | Enable extended header copy size of 256 bytes  if this key is set to true at the global level. |

``interface name`` contains the following keys:

|        Key | Type                      | Notes                                                                                                                                                                                      |
|------------|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| sflow_enable | boolean: true, false*   | Enables sflow at the interface level.  | 
| ingress_enable | boolean: true, false* | Enables ingress sflow at the interface level.  | 
| polling_interval | integer | Configures interface level default counter polling-interval. It can be specified in the range 15-86400. |
| max_header_size | boolean: true,false* | Enable extended header copy size of 256 bytes  if this key is set to true at the interface level. |  
| sample_rate | integer | Configures interface level default sample-rate. It can be specified in the range 256-8388608. | 

```
Note: Asterisk (*) denotes the default value if none is specified. 
```

Connection Variables
--------------------

Ansible Dell EMC Networking roles require the following connection information to establish 
communication with the nodes in your inventory. This information can exist in
the Ansible group_vars or host_vars directories, or in the playbook itself.



|         Key | Required | Choices    | Description                              |
| ----------: | -------- | ---------- | ---------------------------------------- |
|        host | yes      |            | Hostname or address for connecting to the remote device over the specified ``transport``. The value of this key is the destination address for the transport. |
|        port | no       |            | Port used to build the connection to the remote device. If the value of this key does not specify the value, the value defaults to 22. |
|    username | no       |            | Configures the username that authenticates the connection to the remote device. The value of this key authenticates the CLI login. If this key does not specify a value, the value of environment variable ANSIBLE_NET_USERNAME is used instead. |
|    password | no       |            | Specifies the password that authenticates the connection to the remote device. If this key does not specify the value, the value of environment variable ANSIBLE_NET_PASSWORD is used instead. |
|   authorize | no       | yes, no*   | Instructs the module to enter privileged mode on the remote device before sending any commands. If this key does not specify the value, the value of environment variable ANSIBLE_NET_AUTHORIZE is used instead. If not specified, the device attempts to execute all commands in non-privileged mode.|
|   auth_pass | no       |            | Specifies the password to use if required to enter privileged mode on the remote device. If ``authorize`` is set to no, then this key is not applicable. If this key does not specify the value, the value of environment variable ANSIBLE_NET_AUTH_PASS is used instead. |
|   transport | yes      | cli*       | Configures the transport connection to use when connecting to the remote device. This key supports connectivity to the device over CLI (SSH).  |
|    provider | no       |            | Convenient method that passes all of the above connection arguments as a dictonary object. All constraints (such as required, choices) must be met either by individual arguments or values in this dictonary. |


```
Note: Asterisk (*) denotes the default value if none is specified.
```

Dependencies
------------

The dellos-sflow role is built on modules included in the core Ansible code.
These modules were added in Ansible version 2.2.0.

Example Playbook
----------------

The following example uses the dellos.dellos-sflow role to configure sflow attributes at interface and global level.
The example creates a ``hosts`` file with the switch details and corresponding 
variables.
It writes a simple playbook that only references the dellos-sflow role. 
By including the role, you automatically get access to all of the tasks to configure sflow
features. 


Sample ``hosts`` file:
 
    leaf1 ansible_host= <ip_address> ansible_net_os_name= <OS name(dellos9)>

Sample ``host_vars/leaf1``:

    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx 
      password: xxxxx
      authorize: yes
      auth_pass: xxxxx 
      transport: cli

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

 
Simple playbook to setup sflow, ``leaf.yaml``:

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-sflow

Then run with:

    ansible-playbook -i hosts leaf.yaml

License
--------

Copyright (c) 2016, Dell Inc. All rights reserved.
 
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
 
    http://www.apache.org/licenses/LICENSE-2.0
 
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

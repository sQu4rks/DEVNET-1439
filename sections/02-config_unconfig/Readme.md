# Automatic config and unconfig creation with pyATS

pyATS can leverage it's parsers not only to convert a textual configuration, like the output of a `show ip interface` command, to machine-readable data but also do the reverse of that. Based on a python-object we can generate the configuration commands required without writing them ourself. What's more is that pyATS provides the commands to undo the configuration we have just generated. This is commonly referred to in pyATS as the *unconfiguration*. 

1. Create a directory called `02-config_unconfig`. In it, we'll need two files. The `testbed.yaml` file contains the connection details to our devices and the `conf_unconf.py` file will contain the code. Your directory structure should look like this
```
|- 02-config_unconfig/
  |- testbed.yaml
  |- conf_unconf.py
```
2. For our testbed file we'll use connection details that work with our always-on sandbox provided by the Cisco DevNet Sandboxes team. Open your `testbed.yaml` file and copy the following details:

```yaml
---
testbed:
  name: alwaysonsbxs
  credentials: 
    default:
      username: "developer"
      password: "C1sco12345"
      enable: "C1sco12345"

devices:
  csr1000v-1:
    os: iosxe
    type: iosxe
    connections:
      defaults:
        class: unicon.Unicon
      ssh:
        protocol: ssh
        ip: "sandbox-iosxe-latest-1.cisco.com"
        port: "22"
```
3. Next, open the `conf_unconf.py` file. We'll start by importing our loading libraries to read the testbed and also the base object to configure our interface. This is the python class pyATS uses to represent an interface configuration.

```python
from pyats.topology.loader import load
from genie.conf.base import Interface
```
4. Next, we'll load our testbed and connect to the device that we want to configure our interface on. Note that you'll have to change the name of the device if you are using your own testbed file.
```python
testbed = load('testbed.yaml')
uut = testbed.devices['csr1000v-1']
uut.connect(log_stdout=False)
```
5. Finally, we can create an `Interface` object and fill in the interface configuration details such as the IPv4 address, netmask and the switchport configuration.
```python
intf = Interface(device=uut, name="GigabitEthernet5")
intf.ipv4 = "10.10.28.1"
intf.ipv4.netmask = "255.255.255.0"
intf.switchport_enable = False
intf.shutdown = False
```
6. With our `Interface` object done we can use it to generate (and then print out) the configuration and unconfiguration commands. Notice the `apply=False` flag. This tells pyATS to render the configuration but not actually go out to the device and apply it. 
```python
print("Configuration")
print(intf.build_config(apply=False))
print("Un-Configuration")
print(intf.build_unconfig(apply=False))
```

<div align="right">
   
   [Previous](../01-ssot_gitlab_netbox/) - [Next](../03-pyats_bgp/)
</div>
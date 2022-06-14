# Single Source of Truth using GitLab, Templates and Netbox

A single source of truth, or SSoT, provides you with a single store of knowledge. Ideally, **all** your configuration information reside in this repository. 

For this example we create our SSoT by using two different tools. We group devices by families. All your access switches that should be configured the same way are part of the same family and thus share the same *configuration template*. All the device-specific information is then stored in an IPAM system. Here, each device has an entry and a *configuration context* that provides defice-specfic information that can then be used to fill in the placeholders in the configuration template.

For our IPAM system we are using [NetBox](https://docs.netbox.dev/en/stable/). Please refer to their website for information on how to setup an instance. Inside of Netbox, create a device called `router-1` and add a tag called `router` to it. You'll also want to add a local context to that device that contains the message of the day variable we'll be using. You can use the following json as the device-specific local context. 

```
{
    "motd": "This is my motd from Netbox"
}
```

1. Let's start by creating a folder called `templates` that will store our configuration templates. We'll also need two python files. One called `utils.py` which will contain some utility functions and one called `compile_config.py` which will do the actual work of compiling our configuration. We'll also need a json file that will contain the mapping between device tags and configuration templates. Your directory structure should look like this:
```
- 01-ssot_gitlab_netbox
  |- utils.py
  |- compile_configs.py
  |- mapping.json
  |- templates/
```
2. In our `utils.py` file, create the following function that we can use to filter our devices. In this example we filter based on tags.

```python
def get_devices(netbox, tag):
    return netbox.dcim.get_devices(tag=tag)
```
3. Next, we'll associate `tags` with `configuration templates`. That means that, based on the tag of a device, the corresponding template will be used. To feed this information into our system we need a mapping. Create a file called `mapping.json` with the following content:

```json
{
    "router": "router.tpl.conf"
}
```
5. The final step before being able to write our compile script is to add our config templates. These templates are based on [jinja](https://jinja.palletsprojects.com/en/3.1.x/), a very flexible templating language. In your templates folder, create a subfolder called `components`. We'll split our configuration template into reusable components (for example one component for specifying general device configuration information such as a message of the day) that will be pulled in by our router configuration. Your directory structure should now look like this:
```
- 01-ssot_gitlab_netbox
  |- utils.py
  |- compile_configs.py
  |- mapping.json
  |- templates/
    |- components/
```
6. Inside of the components folder, create a file called `motd.tpl.conf` with the following content. This specifies the command for configuring a message of the day as well as a placeholder called `motd` that will be defice-specific and retrieved from Netbox.
```
banner motd $ {{ motd }} $
```
7. With our component done we'll now create the template for our router configuration that will consume the component we just created. In your `templates` folder, create a file called `router.tpl.conf`. Your directory structure should now look like this:
```
- 01-ssot_gitlab_netbox
  |- utils.py
  |- compile_configs.py
  |- mapping.json
  |- templates/
    |- components/
      |- motd.tpl.conf
    |- router.tpl.conf
```
8. Note that the name of your configuration needs to match the name of the configuration file you have associated with a tag in your `mapping.json` file. Inside of the `router.tpl.conf` file we'll simply import our component for now.
```
{% include 'components/motd.tpl.conf' %}
```
9. With the preparations done we can open up our `compile_config.py` file. We start by importing our libraries:
```python
import json
import sys
import os 

import jinja2

from netbox import NetBox
from pathlib import Path

from utils import get_devices
```
10. Next, we setup our connection to netbox and create a jinja2 environment. **Note:** For this example, the connection details are hard-coded into the scripts. This is fine for a demonstration but should **never** be used in production. For production please use environment variables.
```python
netbox = NetBox(host='127.0.0.1', port=8000, use_ssl=False, auth_token='0123456789abcdef0123456789abcdef01234567')
loader = jinja2.FileSystemLoader(searchpath="templates")
env = jinja2.Environment(loader=loader)
```
11. Next, we retrieve the target directory from our command-line arguments, make sure it exists, and load our mappings. 
```python
TARGET_DIR = sys.argv[1]
Path(TARGET_DIR).mkdir(parents=True, exist_ok=True)

conf_mapping = json.load(open("mapping.json"))
```
12. With this setup done we can now loop over all our tags that are known in our mapping, retrieve all devices from netbox that have that tag, load the template associated with the tag and then compile a template for each of the devices in that tag. To do so, we retrieve the device specific `config_context` from netbox and pass it into jinja2. Finally, we write the newly created config to our `TARGET_DIR` folder using the device name as the name of our configuration file. 
```python
for tag, template in conf_mapping.items():
    devices = get_devices(netbox, tag)

    # Load template
    tpl = env.get_template(template) 

    # Compile a config for each device
    for dev in devices:
        # Create context
        ctx = {}
        for k, v in dev['config_context'].items():
            ctx[k] = v

        out = tpl.render(**ctx)        
        
        target_file = os.path.join(TARGET_DIR, f"{dev['name']}.conf")
        with open(target_file, "w") as f:
            f.write(out)
```
13. You can now try and run this script by using the following command:
```
$ python3 compile_config.py TEST
```

This will create a folder called `TEST` in your main directory and in it you should find a compiled template using the configuration context that you specified in Netbox.

<div align="right">
   
   [Back to Overview](../../) - [Next](../02-config_unconfig/)
</div>
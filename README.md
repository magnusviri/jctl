# jctl

This is a Python 3 utility for maintaining & automating Jamf Pro patch management via command-line. The idea behind it is to have a class that maps directly to the Jamf API (https://example.com:8443/api). The API class doesn't abstract anything or hide anything from you. It simply wraps the url requests, authentication, and converts between python dictionaries and xml. It also prints json.

## Requirements

The jctl project requires python3 and requests library. Please make sure you have those by running the following commands.

```bash
$> python3
```

```python
import requests
```

macOS does not include python3. You can get python3 with anaconda or homebrew.

## Installation

In a new shell:

```
$> cd scripts
$> sudo install.py
$> echo 'export PYTHONPATH=/Library/Python/3.6/site-packages' >> ~/.bash_profile
```

If you're using zsh use ~/.zshenv instead of ~/.bash_profile.

If you have /usr/local/bin/plistlib.py make sure it is the python 3 version.

## Authentication

### Authentication Setup

Run the following command to setup or fix the authentication property list file. Note: 8443 is automatically added to the hostname so don't include it.

```$ patch.py config
JSS Hostname: [JAMF PRO HOSTNAME]
username: [USERNAME]
Password: [PASSWORD]
```

The username and password provided will have to be added and given the appropriate access rights.

### Troubleshooting Authentication Setup

The above command should reset the authorization property list, but if you have issues with it not working properly delete the property list file and run the command above again.

`rm ~/Library/Preferences/edu.utah.mlib.jamfutil.plist`

### Authenication File Obfuscation

The authorization property list is obfuscated and encrypted based upon the hostname of the Jamf Pro server and user credentials.

### View Authentication File

To view what is stored in this property list, you could use a command-line tool like `cat`, `less`, etc.

For example...

```
$ cat ~/Library/Preferences/edu.utah.mlib.jamfutil.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Credentials</key>
	<data>
	YnBsaXN0MDSRAQRfEBNjYXNwZXIuc2NsLnV0YWguZWR1TxBCH/k+SqHI7doBuB0l/GIS
	4onl2JLjVwjkMFax1+6YgrEUaYlSI9K83euiuR99iVIj0r/d6qO5H32JUiPSvN3qo7kA
	CAshAAAAAAAAAQEAAAAAAAAAAwAAAAAAAAAAAAAAACAAAGY=
	</data>
	<key>JSSHostname</key>
	<string>jamf.domain.edu</string>
</dict>
</plist>
```

## Using the API

The API script interacts with Jamf using the get, post, put, and upload commands in combination with the API resources. To see all of your resources, go to the following URL on your server. https://example.com:8443/api

The api can be interacted with via python3 shell. This is how you set it up.

```bash
$> cd jctl
$> python3
```

```python
from pprint import pprint
import logging
import jamf

fmt = '%(asctime)s: %(levelname)8s: %(name)s - %(funcName)s(): %(message)s'
logging.basicConfig(level=logging.DEBUG, format=fmt)
logger = logging.getLogger(__name__)

# create an jamf.API object (requires requests lib)
logger.debug("creating api")
jss = jamf.API()
```

### Examples of getting data.

Note: The API get method downloads the data from Jamf. If you store it in a variable, it does not update itself. If you make changes on the server, you'll need to run the API get again.

```python
# Get any information from your jss using the classic api endpoints. This includes nested dictionaries.

pprint(jss.get('accounts'))
pprint(jss.get('buildings'))
pprint(jss.get('categories'))
pprint(jss.get('computergroups'))
pprint(jss.get('computers'))
pprint(jss.get('departments'))
pprint(jss.get('licensedsoftware'))
pprint(jss.get('networksegments'))
pprint(jss.get('osxconfigurationprofiles'))
pprint(jss.get('packages'))
pprint(jss.get('patches'))
pprint(jss.get('policies'))
pprint(jss.get('scripts'))

# Get all categories (and deal with the nested dictionaries)

categories = jss.get('categories')['categories']['category']
category_names = [x['name'] for x in categories]
print(f"first category: {category_names[0]}")
pprint(category_names)

# Get computer management information (this demonstrates using an id in the get request)

computers = jss.get('computers')['computers']['computer']
pprint(computers[0])
pprint(jss.get(f"computermanagement/id/{computers[0]['id']}"))
pprint(jss.get(f"computermanagement/id/{computers[0]['id']}/subset/general"))

# Getting smart computer groups using list comprehension filtering.

computergroups = jss.get('computergroups')['computer_groups']['computer_group']
smartcomputergroups = [i for i in computergroups if i['is_smart'] == 'true']
pprint(smartcomputergroups)
staticcomputergroups = [i for i in computergroups if i['is_smart'] != 'true']
pprint(staticcomputergroups)
computergroupids = [i['id'] for i in computergroups]
pprint(computergroupids)
```

### Example of posting data.

```python
# Create a new static computer group. Note, the id in the url ("1") is ignored and the next available id is used. The name in the url ("ignored") is also ignored and the name in the data ("realname") is what is actually used.
import json
jss.post("computergroups/id/1",json.loads( '{"computer_group": {"name": "test", "is_smart": "false", "site": {"id": "-1", "name": "None"}, "criteria": {"size": "0"}, "computers": {"size": "0"}}}' ))
jss.post("computergroups/name/ignored",json.loads( '{"computer_group": {"name": "realname", "is_smart": "false", "site": {"id": "-1", "name": "None"}, "criteria": {"size": "0"}, "computers": {"size": "0"}}}' ))
```

### Examples of updating data.

```python
# Create a new static computer group. Note, the id ("1") is ignored and the next available id is used.

import json
jss.put("computergroups/name/realname",json.loads( '{"computer_group": {"name": "new name", "is_smart": "false", "site": {"id": "-1", "name": "None"}, "criteria": {"size": "0"}, "computers": {"size": "0"}}}' ))
jss.put("computergroups/id/900",json.loads( '{"computer_group": {"name": "newer name", "is_smart": "false", "site": {"id": "-1", "name": "None"}, "criteria": {"size": "0"}, "computers": {"size": "0"}}}' ))
```

### Examples of deleting data.

```python
jss.delete("computergroups/name/new name")
jss.delete("computergroups/id/900")
```

## Using the other classes


### Examples of 

from jamf.category import Categories

### Running Tests

```bash
$> cd jctl

# runs all tests
$> python3 -m unittest discover -v

# run tests individually
$> python3 -m jamf.tests.test_api
$> python3 -m jamf.tests.test_config
$> python3 -m jamf.tests.test_convert
$> python3 -m jamf.tests.test_package
```

If you see an error that says something like SyntaxError: invalid syntax, check to see if you're using python3.

### Getting Help
```
$> patch.py --help
$> patch.py list --help
$> patch.py upload --help
$> patch.py config --help
$> patch.py remove --help
$> patch.py info --help
$> patch.py update --help
```

### List all Patch Management Title Names
```$> patch.py list```

### List all uploaded packages
`$> patch.py list --pkgs`

### List all versions (and associated packages)
```
$> patch.py list --versions <Name of Patch Management Title>
$> patch.py list --patches <Name of Patch Management Title>
```

### Modify Patch settings

The following requires the user to have Jamf Admin Privileges

```
$> patch.py info /PATH/TO/PACKAGE
$> patch.py upload /PATH/TO/PACKAGE
$> patch.py remove <PACKAGE NAME>
```

### Packages

As of `1.1` Policy packages can be modified using the following:

```python
import jamf

# create an jamf.API object (requires requests lib)
logger.debug("creating api")
jss = jamf.API(config='private/jss.plist')

policy = jamf.Policy(jss, name="Policy Name")

# add a package named "example.pkg" to the policy (must exist in JSS)
policy.add_package("example.pkg")

# remove "example.pkg" from the policy
policy.remove_package("example.pkg")

# add "example.pkg" to be cached
policy.add_package("example.pkg", action="Cache")

# remove all packages for policy
policy.remove_all_packages()

```

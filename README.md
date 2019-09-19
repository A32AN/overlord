# Overlord – Red Teaming Automation

- [Overlord – Red Teaming Automation](#overlord-%e2%80%93-red-teaming-automation)
- [Setup](#setup)
- [Documentation](#documentation)
  - [Projects](#projects)
  - [Supported Providers](#supported-providers)
  - [Variables](#variables)
  - [Modules](#modules)
    - [c2](#c2)
    - [dns_records](#dnsrecords)
      - [Type](#type)
      - [Record](#record)
      - [Name](#name)
    - [gophish](#gophish)
    - [letsencrypt](#letsencrypt)
    - [mail](#mail)
    - [redirector](#redirector)
    - [webserver](#webserver)
    - [godaddy](#godaddy)
- [Arguments](#arguments)
  - [Help](#help)
  - [Advanced Configuration](#advanced-configuration)
    - [Installation Templates](#installation-templates)
    - [Default Configuration File](#default-configuration-file)
- [RedBaron](#redbaron)
  - [Notes](#notes)
    - [Firewall rules](#firewall-rules)


This tool provides a python-based console CLI which is used to build Red Teaming infrastructure in an automated way. The user has to provide inputs by using the tool’s modules (e.g. C2, Email Server, HTTP web delivery server, Phishing server etc.) and the full infra / modules and scripts will be generated automatically on a cloud provider of choice. Currently supports AWS and Digital Ocean. The tool is still under development and it was inspired and uses the [Red-Baron](https://github.com/byt3bl33d3r/Red-Baron) Terraform implementation found on Github. 

It was only tested on Kali Linux but it probably work on all Linux x64 systems.
# Setup

```bash
git clone https://github.com/qsecure-labs/overlord.git /opt/
cd /opt/overlord/config/overlord
./install.sh
```
# Documentation
## Projects
Overlord has build in functionality for project management. From the cli you can manage each project by loading the configuration file with the `load` command. When you deploy the project again, the modifications will be pushed to the providers. For more information visit the [Help](#help).
## Supported Providers
  - Digital Ocean
  - AWS
  - Godaddy
## Variables
The `set` command can be used to initialize the API keys to communicate with the providers. The  domains variable can be used  to add domain names into the overlord project.
```
aws_access_key        aws_secret_key        domains               dotoken               godaddy_access_key    godaddy_secret_key
```
The `./projects/variables.json` can be used to auto load the keys used to authenticate with each of the providers overlord supports and the domain names. When you first set the arguments into your campaign you can save them using the `set variables` command which will create the `variables.json` file. 
## Modules
### c2
Creates a C2 server of the provider of choice in the cloud. The types available are HTTP/DNS. SSH keys for each instance will be outputted to the ```redbaron/data/ssh_keys``` folder.

|Variable   	|Required   |Description   	                                            |
|---	        |---	    |---	                                                        |
|`id` 	      |N/A   	    |Module ID Autogenerated        	                          |
|`type`   	  |Yes   	    |Type of c2 Accepted values are: HTTP/DNS.   	              |
|`provider`   |Yes   	    |Provider to be used   	                                    |
|`region`   	|Yes   	    |Regions to create server instance   	                      |
|`size`   	  |Yes   	    |Instance size to launch   	                                |
|`redirectors`|Yes   	    |Number of redirectors to launch for each c2. It can be 0.  |
|`tools`   	  |No   	    |Tools to be installed on instance creation.   	            |

The tools that are currently embedded to be automatically install are the following:
- metasploit
- empire
- dnscat2
- cobaltstrike (The `CSTRIKE_KEY` variable has to be set in the  `./redbaron/data/scripts/tools/cobaltstrike.sh` script)
- The PenTesters Framework `(PTF)` (A library of penetration testing tools. You can modify what  you  want to install by changing the  `./redbaron/data/scripts/tools/ptf.sh` script. For more information about  the  project visit: https://github.com/trustedsec/ptf)

### dns_records
Adds records to a domain.

|Variable   	  |Required   |Description   	                                            |
|---	          |---	      |---	                                                      |
|`id` 	        |N/A   	    |Module ID Autogenerated        	                          |
|`provider`   	|Yes   	    |Provider to be used   	                                    |
|`type`   	    |Yes   	    |The record type to add.                                    |
|`record`   	  |Yes   	    |The record to add. See record section.                     |
|`name`   	    |Yes   	    |Name of the subdomain   	                                  |
|`priority`   	|No   	    |Used for mail server. Default 1.                           |
|`ttl`       	  |No   	    |Time to live                                 	            |

#### Type
Valid values are A, MX and TXT.
#### Record
The record to add.
```
A:   set record -m <module_id> -d <domain>  
TXT: set record -d <domain> -t <template>
TXT: set record -d <domain> -v <custom>
MX:  set record -m <module_id> -d <domain>
```
#### Name
Use '@' for DigitalOcean or "" to AWS to create the record at the root of the domain or enter a hostname to create it elsewhere.A records are for IPv4 addresses only and tell a request where your domain should direct to. 

### gophish
Creates a gophish server of the provider of choice in the cloud. SSH keys for each instance will be outputted to the ```redbaron/data/ssh_keys``` folder.

|Variable   	  |Required   |Description   	                                            |
|---	          |---	      |---	                                                      |
|`id` 	        |N/A   	    |Module ID Autogenerated        	                          |
|`provider`   	|Yes   	    |Provider to be used   	                                    |
|`region`   	  |Yes   	    |Regions to create server instance   	                      |
|`size`   	    |Yes   	    |Instance size to launch   	                                |
|`redirectors`  |Yes   	    |Number of redirectors to launch for each c2. It can be 0.  |

### letsencrypt
Creates a Let's Encrypt TLS certificate for the specified domain using the DNS challenge. It stores the certificates on the ```redbaron/data/certificates``` or if it is a web server it runs certbort on the server.

|Variable   	|Required   |Description   	                                            |
|---	        |---	      |---	                                                      |
|`id` 	      |N/A   	    |Module ID Autogenerated        	                          |
|`domain_name`|Yes   	    |The domain name for the certificate                        |
|`email`   	  |Yes   	    |Email for certificate defaults to kokos@example.com        |
|`mod_id`     |No   	    |Autoloaded from domain_name                                |

### mail
Creates a mail server of the provider of choice in the cloud. SSH keys for each instance will be outputted to the ```redbaron/data/ssh_keys``` folder.

|Variable   	  |Required   |Description   	                                            |
|---	          |---	      |---	                                                      |
|`id` 	        |N/A   	    |Module ID Autogenerated        	                          |
|`domain_name`  |Yes   	    |Domain Name to use.   	                                    |
|`subdomain`   	|Yes   	    |Subdomain to use.                                          |
|`allowed_ips`  |Yes   	    |IPs which are allowed to connect to relay emails.          |
|`provider`   	|Yes   	    |Provider to be used   	                                    |
|`region`   	  |Yes   	    |Regions to create server instance   	                      |
|`size`   	    |Yes   	    |Instance size to launch   	                                |

### redirector
Creates a redirector server for another module of the provider of choice in the cloud. The types availalbe are HTTP/DNS. SSH keys for each instance will be outputted to the ```redbaron/data/ssh_keys``` folder.

|Variable   	  |Required   |Description   	                                            |
|---	          |---	      |---	                                                      |
|`id` 	        |N/A   	    |Module ID Autogenerated        	                          |
|`type`   	    |Yes   	    |Type of c2 Accepted values are: HTTP/DNS.   	              |
|`provider`   	|Yes   	    |Provider to be used   	                                    |
|`region`   	  |Yes   	    |Regions to create server instance   	                      |
|`size`   	    |Yes   	    |Instance size to launch   	                                |
|`redirector_id`|Yes   	    |ID of the redirector to set up.                            |

### webserver
Creates a web server of the provider of choice in the cloud. SSH keys for each instance will be outputted to the ```redbaron/data/ssh_keys``` folder.

|Variable   	|Required   |Description   	                                            |
|---	        |---	    |---	                                                        |
|`id` 	      |N/A   	    |Module ID Autogenerated        	                          |
|`provider`   |Yes   	    |Provider to be used   	                                    |
|`region`   	|Yes   	    |Regions to create server instance   	                      |
|`size`   	  |Yes   	    |Instance size to launch   	                                |
|`redirectors`|Yes   	    |Number of redirectors to launch for each c2. It can be 0.  |

### godaddy
Redirects the nameservers from Godaddy to another provider (AWS, DigitalOcean)

# Arguments

| Name                      | Required | Description
|---------------------------| -------- | -----------
|`id`	                      |N/A	     |Module ID Autogenerated 	              |
|`provider`   	            |Yes   	   |Provider to be used   	                |
|`domain`                   | Yes      | The domain to create a hosted zone for.|

## Help
The help menu can provide additional information about each command.
```
Overlord$> help -v

Documented commands (type help <topic>):

General (type help <command>)
================================================================================
info                Prints variable table or contents of a module which was added to the campaign
set                 General variables for the campaign to be set

Module  (type help <command>)
================================================================================
delmodule           Deletes a module
editmodule          Edits a module
usemodule           Usemodule command help

Project (type help <command>)
================================================================================
create              Creates terraform project from the campaign
delete              Deletes a project
deploy              Deploy current  project
load                Load a project to overlord
new                 Creates new terraform project.
rename              Rename a project
save                Save a project

Other
================================================================================
clear               Clear the screen
exit                Exit to main menu
help                List available commands or provide detailed help for a specific command
history             View, run, edit, save, or clear previously entered commands
shell               Execute a command as if at the OS prompt
version             Version
```

```
Overlord$> help set 
usage: set [-h] {dotoken,aws_secret_key,aws_access_key,domains,variables} ...

General variables for the campaign to be set

optional arguments:
  -h, --help            show this help message and exit

set-commands:
  {dotoken,aws_secret_key,aws_access_key,domains,variables}
                        set-command help
    dotoken             Sets the Digital Ocean Token
    aws_secret_key      Sets the AWS Secret Key
    aws_access_key      Sets the AWS Access Key
    domains             Domain names to be used in the campaign (Multiple domain names can be added)
    variables           Sets the default variables.json to the values that are in memory
```

```
Overlord$> help set dotoken 
usage: set dotoken [-h] dotoken

positional arguments:
  dotoken     example : [ set dotoken <token>]

optional arguments:
  -h, --help  show this help message and exit
```
## Advanced Configuration

### Installation Templates
In the c2 module the user has the ability to install tools from a list. The tool automatically loads the scripts from the `./redbaron/data/scripts/tools`. By adding a new script in the directory, you can install by the `tools` variable in the c2 module.

### Default Configuration File
The `./config/config.json` file contains the default configuration on each module and the providers that are used by overlord. By changing each object, you can customize the default values of each module when it loads to overlord.
```json
    // Default
    "mod_c2": {
        "module": "c2",
        "type" : "http",
        "redirectors": 1,
        "tools": [],
        "region": "LON1",
        "provider": "digitalocean",
        "size": "s-1vcpu-1gb",
        "id": ""
    }

    // Customized
    "mod_c2": {
        "module": "c2",
        "type" : "http",
        "redirectors": 0,
        "tools": ["metasploit","empire"],
        "region": "us-east-2",
        "provider": "aws",
        "size": "t2.nano",
        "id": ""
    }
```

# RedBaron
**Red Baron only supports Terraform version 0.11 and will only work on Linux x64 systems. We will try to update it to support newer versions as well on a later date.** 

For more information on how to modify the terraform modules and about the Red Baron project visit the following [Link](https://github.com/byt3bl33d3r/Red-Baron).

## Notes
### Firewall rules
Overlord does not support adding new firewall rules from the CLI at the current time. You can add or remove the rules  from the RedBaron modules directory on the  terraform code or after the installation on each provider.
- AWS: https://www.terraform.io/docs/providers/aws/r/security_group.html
- DIGITALOCEAN: https://www.terraform.io/docs/providers/do/r/firewall.html

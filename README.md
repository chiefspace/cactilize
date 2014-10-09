# Cactilize

Ansible playbooks for deploying and maintaining cacti server and host (graph, devices, templates, tree ...)

* [Definitions](#definitions)
* [Description](#description)
* [Requirements](#requirements)
* [Installation](#installation)
* [QuickStart](#QuickStart)
  - [Install and bootstrap an cacti server instance](#Install and bootstrap an cacti server instance)
  - [Tips](#Tips)
* [Development](#development)
* [Documentation](#documentation)
* [Credits](#credits)
* [Licence](#licence)
* [References](#references)

## Definitions

In the following :

 - The **Server** is the host where **cacti server** is installed. Cacti is a complete network graphing solution designed to harness the power of [*RRDTool's*](http://www.rrdtool.org/) data storage and graphing functionality. Cacti provides a fast poller, advanced graph templating, multiple data acquisition methods, and user management features out of the box. All of this is wrapped in an intuitive, easy to use interface that makes sense for LAN-sized installations up to complex networks with hundreds of devices.
 
	 ![cacti](http://www.cacti.net/images/cacti_promo_main.png)
 
 - The **Cacti client** A device, be anything which can be monitored remotely or locally. This can include storage devices, Windows or UNIX servers, and of course network devices. For Cacti to be able to monitor a device, it needs to be reachable by ping or SNMP, but the actual data retrieval can also be done using scripts and commands, or a set of SNMP queries.


## Requirements

 - [Ansible](http://www.ansibleworks.com) installed and ready to work. If not and you have [puppet](http://puppetlabs.com/) master in you architecure, I highly recommend the [nvogel/ansible](http://forge.puppetlabs.com/nvogel/ansible) Module.

 - MySQL 5.x or greater and ensure you have set a auto-login for root by creating a `/root/.my.cnf` on 0400 mode and put this 3 lines according your mysql server configuration. 

  ```bash
[client]
user=root
host=localhost
password='You mysql server root password'
```

> This role is created for Debian (Squeeze/Wheezy) and compatible with ***ansible 1.6 and ansible 1.7***.


## Installation

From Ansible Galaxy :
```bash
su - ansible
$ ansible-galaxy install helldorado.cactilize
```
Or with [Librarian ansible](https://github.com/bcoe/librarian-ansible), for example add to your **Ansiblefile** :

```
role "cactilize", :git => "https://github.com/helldorado/cactilize"
:ref => '0.1'
```
And update your roles
```bash
$ librarian-ansible update
```

## QuickStart

### Install and bootstrap an cacti server instance

- The ansible host file: `cactilize`


```ini
### cacti server
[server] 
spymedia

### Client to graph
[client]
spyweb[1:8]
spycache[1:3]
spybdd1[1:2]
spywatch1
spymail
```
- The playbook file: `cactilize.yml`


```yaml
# File: cactilize.yml
#
# abdoul.bah@alterway.fr
#
# Description :

# OS: Debian6 debian7

---
- hosts: all
  user: ansible
  sudo: True

  vars_files:
    - roles/cactilize/vars/deploy.yml
    - roles/cactilize/vars/templates.yml
    - roles/cactilize/vars/devices.yml
    - roles/cactilize/vars/users.yml
    - roles/cactilize/vars/trees.yml
    - roles/cactilize/vars/def/mysql_def.yml
    - roles/cactilize/vars/def/elasticsearch_def.yml
    - roles/cactilize/vars/def/system_def.yml
    - roles/cactilize/vars/def/varnish_def.yml
    - roles/cactilize/vars/def/memcache_def.yml
    - roles/cactilize/vars/def/apache_def.yml
    - roles/cactilize/vars/def/redis_def.yml
    - roles/cactilize/vars/def/mongodb_def.yml
    - roles/cactilize/vars/def/galera_def.yml
    - roles/cactilize/vars/def/nginx_def.yml
    - roles/cactilize/vars/def/all_def.yml

  roles:
   - cactilize
 ```
 
- The deploy vars file: `roles/cactilize/vars/deploy.yml`

```yaml
---
### DEPLOIEMENT 
deploy                : false
webui_admin_user      : helldorado
webui_admin_password  : 2A2169234F6BC1360CFC29EEF8
archi_name            : spy
archi_subnet          : '10.0.2'
default_community     : spynol
cacti_db_hostname     : localhost
cacti_db_password     : tNSimlfnER7d6
cacti_mysql_mon_user  : monitoring
cacti_mysql_mon_pass  : 4vtYd5axfavQo
#domain_name          : spynol.info
RRA_VG_NAME           : system
#RRA_LV_NAME          : var_lib_rra
RRA_LV_SIZE           : 5G
RRA_MOUNT_POINT       : '/var/lib/rra'
DEFAULT_IP            : 'x.x.x.0/24'
WHITELIST             : 'x.x.x.1/32 y.y.y.2/32 z.z.z.0/16'

### HTPASSWD USERS
htpasswd_users:
  spyadm:
    passwd: tSuMCNaKvASaU 
  spycombo:
    passwd: YbXpnONCIG9V2
```
- The host (devices) vars file:  `roles/cactilize/vars/hosts.yml`

```yaml
---
Hosts:
  'spyweb1':
    IP   : 10.0.2.11
    graph: 
      - system
      - nginx
      - apache
    tree : LEMP

  'spybdd1':
    IP   : 10.0.2.51
    graph:
      - system
      - mysql
      - memcache
      - redis
      - mongodb
    tree : Mysql
    notes: 'Database Server'
  
  'spycache1':
    IP   : 10.0.2.16
    graph: 
      - system
      - varnish
      - memcache
      - galera
    tree : Varnish
    notes: 'Cache Server'
  
  'spywatch':
    IP   : 10.0.2.40
    graph: 
      - system
      - elasticsearch
      - redis
    tree : ES
    notes: 'Elasticsearch server'
  
  'spymail':
    IP   : 10.0.2.70
    graph: 
      - system
      - postfix
    tree : Mail
    notes: 'Mail Server'
```
- The tree vars file: `roles/cactilize/vars/trees.yml`


```yaml
---

## TREE SERVICE HOST BASED
#tree_mode          : host_by_role
tree_mode         : graph_by_role
#tree_mode          : mixed
parentnode_host    : HOSTS
parentnode_service : SERVICES


## HOST BY ROLES
Tree_Mode_Host:
 
 - node: "{{ parentnode_host }}"
   subnodes:
     -

 - node: LVS
   subnodes:
     -

 - node: WEB
   subnodes:
     - Nginx
     - Apache
     - Litghttpd

 - node: CACHE
   subnodes:
     - Varnish
 
 - node: DATABASES
   subnodes:
     - Mysql
     - Galera
     - Postgresql

 - node: NoSQL
   subnodes:
     - Redis
     - Memcache
     - Mongodb
     - ES
 
 - node: LAMP
   subnodes:
      -

 - node: LEMP
   subnodes:
      -

## GRAPH BY ROLE
Tree_Mode_Graph:

 - node: "{{ parentnode_service }}"
   subnodes:
     -

 - node: WEB
   subnodes:
     - NGINX
     - APACHE
     - LIGHTTPD

 - node: CACHE
   subnodes:
     - VARNISH
     - OPCODE

 - node: DATABASES
   subnodes:
     - MYSQL
     - GALERA

 - node: NoSQL
   subnodes:
     - REDIS
     - MEMCACHE
     - MONGODB
     - ES

 - node: SYSTEM
   subnodes:
     - NETWORK
     - CPU
     - MEMORY
     - DISK
```  

- The users vars file: `roles/cactilize/vars/users.yml`

```yaml
---
 ## WEBUI User Permissions
Users:

  'spyviewer':
    password            : FleninOfAt
    full_name           : 'SPY Viewer'
    enabled             : 'on'
    must_change_password:
    permissions         :
      - View_Graphs
      - Export_Data
    policy              :
      - View_Graphs
      - View_Tree
```



**/!\  For the other vars files, PLEASE ‼️ Do not edit setting unless you know what you are doing**



> You can create several devices, tree and users.


### Ready to fire ? GO ! 

 - Step 1 :: Deploy cacti server 

```bash
ansible-playbook cactilize.yml -i cactilize --limit server -extra-vars deploy=true
```

This will take some time according your devices and graph list, be patient...

- Step 2 :: Configure your client to send information via SNMP, Scripts and others to the cacti server.

```bash
ansible-playbook cactilize.yml -i cactilize --limit client
```

- Step 3 :: check the report file `/root/.cacti` in you cacti server. For example

```ini
-- ************************** WARNING CACTILIZE ANSIBLE PLAYBOOK ********************************
-- File Managed by Ansible
-- Any changes to this report file  should be done trough ansible
-- other wise report will be overwriten on te next ansible cactilize deploy pass.
-- **********************************************************************************************

  /$$$$$$                        /$$     /$$ /$$ /$$
 /$$__  $$                      | $$    |__/| $$|__/
| $$  \__/  /$$$$$$   /$$$$$$$ /$$$$$$   /$$| $$ /$$ /$$$$$$$$  /$$$$$$
| $$       |____  $$ /$$_____/|_  $$_/  | $$| $$| $$|____ /$$/ /$$__  $$
| $$        /$$$$$$$| $$        | $$    | $$| $$| $$   /$$$$/ | $$$$$$$$
| $$    $$ /$$__  $$| $$        | $$ /$$| $$| $$| $$  /$$__/  | $$_____/
|  $$$$$$/|  $$$$$$$|  $$$$$$$  |  $$$$/| $$| $$| $$ /$$$$$$$$|  $$$$$$$
 \______/  \_______/ \_______/   \___/  |__/|__/|__/|________/ \_______/


[ACCESS]
   URL   => http://IP_PUB/cacti , https://IP_PUB/cacti (according your server and archi configuration)

   - WebUI
       Administrator:
          Login  => helldorado
          Pass   => 2A2169234F6BC1360CFC29EEF8

       SPY Viewer:
          Login  => spyviewer
          Pass   => FleninOfAt


/!\ Please for nginx server, put this line in your default virtual host:

include /etc/nginx/conf.d/status.c****onf ;

And reload nginx:

nginx -t && service nginx reload
```


## Tips
- Allways use and ABUSE **TAGs** 

| List of Tags  | Description | File|
| ------------- | ------------- |--------
|   device      | Create all device  | task/devices.yml 
|   graph       | Create all graph |task/graph.yml| 
|   graph-system| Create only system graph|
|   graph-mysql | Create only mysql graph |
|  graph-mongodb|Create only mongodb graph|
- If possible or necessary  start from a specific task 

```bash
ansible-playbook cactilize.yml -i cactilize -l client --start-at-task "SNMP CONF"
``` 

## Development

### Bugs and feature requests

Have a bug or a feature request? Please first check the list of issues.

If your problem or idea is not addressed yet, please open a new issue, or contact me at [devops@helldorado.info](devops@helldorado.info)

### Contributing

You're welcome to propose pull requests. Here's a quick guide.

Fork, then clone the repo:

    git clone git@github.com:your-username/cactilize

Set up your ansible environement for test suite.

Make sure the tests pass:

Make your change. Add examples and documentation for your change. 
Push to your fork and [submit a pull request](https://github.com/helldorado/cactilize/compare/).


Some things that will increase the chance that your pull request is accepted:

* Write tests, see ansible [Testing Strategies](http://docs.ansible.com/test_strategies.html)
* Follow ansible playbook best pratices [style guide]().
* Write a [good commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).




### Branch management


## Credits

* **Abdoul Bah**
* [All contributors]

## Licence

Cactilize ansible role is released under the GPL2 License. Check the LICENSE file for details.

## References

- [ansible](http://www.ansibleworks.com)
- [contributing to open-source](https://guides.github.com/activities/contributing-to-open-source)
- [commit]()

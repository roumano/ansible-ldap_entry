Role Name
=========
Use to create/update ldap account , update ssh key, ...
Actualy, it's more tasks than real roles

Requirements
------------
ansible version >= 2.3
New module : 
ldap_entry : http://docs.ansible.com/ansible/ldap_entry_module.html
ldap_attr http://docs.ansible.com/ansible/ldap_attr_module.html

Dependencies
------------
Need role quinot.lookup_ldap to get information from ldap (used only for UnixAccount to known which uidNumber are already used)

Role Variables
--------------
Add your LDAP credential into a encrypted (recommended) configuration file
Where the vault file conf/${USER}.yml contents :
```
 bind_dn: "uid=XXX,ou=People,dc=XXXX,dc=XXX"
 bind_pw: "XXXX"
```
* Optional :
create this variable : nmcli_name: "OpenVpn_Name"
in order to use network manager command line (nmcli) to open the openvpn
Use it on ansible via : 
```
ansible-playbook ... --extra-vars @conf/${USER}.yml --ask-vault
```

Example Playbook
----------------

## WebPeople
enable it with job: webaccount
```
---
- name: Create WebAccount
  hosts: localhost
  gather_facts: no
  vars:
    - job: webaccount
    - mail: christian.iuga@edifixio.com
    - description: "{{ ansible_date_time.date }}: create account for {{ mail }}"
    - group_name: 'cn=wiki'
  roles:
    - ansible-ldap_entry
```
or you can create a configuration file and load (vars_file or  --extra-vars @file.yml)

| variable | mandatory |default | description |
|:--------:|:---------:|:------:|:------------|
| sn | no | sn in mail via regexp sn.XXXX@XXX.XXX | first name of the person. |
| cn | no | cn in mail via regexp XXX.cn@XXX.XXX | lastname of the person. |
| uid | no | first letter of sn + cn | account name. |
| mail | yes | | mail adress of the person. |
| description | yes | | format wanted : 2017/01/19: TS #9999 create account. |
| group | no | | if "True" add the uid into cn=group,ou=Group... |

```
ansible-playbook site.yml --extra-vars @conf/webpeople.yml --extra-vars @conf/${USER}.yml --ask-vault
```
with file conf/webpeople.yml :
```
uid: test10
sn: FirstName
cn: CommonName
mail: christian.XXX@XXXX.com
description: "2017/01/19 Ticket #9999 create account {{ uid }}"
```

## UnixAccount
enable it with job: account

```
- name: Create UnixAccount
  hosts: localhost
  gather_facts: yes
  vars:
  - job: account
  vars_files:
  - conf/people.yml
  roles:
  - ansible-ldap_entry
```
As need lot of variable, it's better to work from a variable file
see sample files for exemples

| variable | mandatory |default | description |
|:--------:|:---------:|:------:|:------------|
| sn | no | sn in mail via regexp sn.XXXX@XXX.XXX | first name of the person. |
| cn | no | cn in mail via regexp XXX.cn@XXX.XXX | lastname of the person. |
| uid   | no | first letter of sn + cn | account name. |
| mail | yes | | mail adress of the person. |
| compagny | no | regexp in mail XXX@compagny.XXX | used to fil gecos information. |
| description | yes | | format wanted : 2017/01/19: TS #9999 create account. |
| uidNumber | no | first uidNumber not used in range [ 5000 - 5100 ]| uidNumber of the account. |
| gidnumber | no | uidNumber | gidnumber of the account. |
| host | no | | server permit to a ssh access. |
| loginShell | no | /bin/bash | shell used when connexion to a server. |
| homeDirectory | no | /home/{{ uid }} | homedirectory of the account. |
| sshPublicKey | no | | ssh public Key of the account. |
| group_access | no | | if "True" add the uid into cn=group,ou=Group. |

```
ansible-playbook site.yml
ansible-playbook site.yml --extra-vars @conf/people.yml --extra-vars @conf/${USER}.yml --ask-vault
```

## SSH Key
enable it with job: ssh

```
- name: Update ssh key
  hosts: localhost
  gather_facts: yes
  vars:
  - job: ssh
  - ou_people: "ou=FR,ou=Admins,ou=People"
  vars_files:
  - conf/admin_user2.yml
  roles:
  - role: ansible-ldap_entry
    users: "{{ admin_users }}"
```

Where the file contents a dictonnary of account with : (same syntax as cchurch.admin-users roles
```
admin_users:
  - fullname: 'FirstName LastName'
    username: 'uid'
    pubkey: 'ssh-rsa AAAAB3NXXXXX'
    exclusive: true
  - fullname: '....'
    ...
```

Note
----
if want to use {{ ansible_date_time.date }} in description, it's mandatory to use gather_facts: yes

Known issue :
- SSH module : TODO better detect non existing user
  actualy add a simple failed_when: true
  "msg": "Cannot search for attribute sshPublicKey" when the user don't exist
- Unix Account : uid range is hardcode in task "create array or uidRange"

License
-------

GPL


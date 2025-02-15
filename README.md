# DEPRECATED

Using UFW as middleware in automation does not make real sense.

It creates unnecessary complexity for single-rule changes!

In my eyes it is not a tool that is designed to be automated.

I would actually recommend using NFTables: [ansibleguy.infra_nftables](https://github.com/ansibleguy/infra_nftables) 

# Ansible Role - Uncomplicated Firewall (UFW)

Ansible Role to deploy/configure the software firewall 'UFW' on a debian-based linux server.

<a href='https://ko-fi.com/ansible0guy' target='_blank'><img height='35' style='border:0px;height:46px;' src='https://az743702.vo.msecnd.net/cdn/kofi3.png?v=0' border='0' alt='Buy me a coffee' />

[![Molecule Test Status](https://badges.ansibleguy.net/linux_ufw.molecule.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/molecule.sh.j2)
[![YamlLint Test Status](https://badges.ansibleguy.net/linux_ufw.yamllint.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/yamllint.sh.j2)
[![PyLint Test Status](https://badges.ansibleguy.net/linux_ufw.pylint.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/pylint.sh.j2)
[![Ansible-Lint Test Status](https://badges.ansibleguy.net/linux_ufw.ansiblelint.svg)](https://github.com/ansibleguy/_meta_cicd/blob/latest/templates/usr/local/bin/cicd/ansiblelint.sh.j2)
[![Ansible Galaxy](https://badges.ansibleguy.net/galaxy.badge.svg)](https://galaxy.ansible.com/ui/standalone/roles/ansibleguy/linux_ufw)

**Tested:**
* Debian 11

## Install

```bash
# latest
ansible-galaxy role install git+https://github.com/ansibleguy/linux_ufw

# from galaxy
ansible-galaxy install ansibleguy.linux_ufw

# or to custom role-path
ansible-galaxy install ansibleguy.linux_ufw --roles-path ./roles

# install dependencies
ansible-galaxy install -r requirements.yml
```

## Functionality

This ansible role will do:
* **Package installation**
  * UFW


* **Configuration**
  * Rules via using **one of two modes**
    * The **stateful** way (_default_)
      * keeps existing rules and adds/removes rules using a rule state
    * The **stateless** way
      * reset's the ufw state and rules every time
      * after that the new rules get applied


  * Verification that a ssh-rule is in place


## Info

* **Note:** Most of the role's functionality can be opted in or out.

  For all available options - see the default-config located in [the main defaults-file](https://github.com/ansibleguy/linux_ufw/blob/latest/defaults/main.yml)!


* **Note:** this role currently only supports debian-based systems


* **Warning:** Not every setting/variable you provide will be checked for validity. Bad config might break the role!


## Usage

You want a simple Ansible GUI? Check-out my [Ansible WebUI](https://github.com/ansibleguy/webui)

### Config

Just define the 'ufw_rules' dictionary as needed:
```yaml
ufw_rules:
  ruleShortName:
    rule: 'allow'  # default if empty
    port: 80
    proto: 'tcp'
    log: 'no'  # default if empty
    from_ip: 'any'  # default if empty
    to_ip: 'any'  # default if empty
    direction: 'in'  # default if empty
    present: true  # default if empty => will be used for stateful rule-check (alias = state: present)
    position: 2  # you can define the position of the rule in the ruleset (alias = insert)
    comment: 'You can overwrite the default comment'
```
or the compact way:
```yaml
ufw_rules: {
    ruleShortName: {rule: 'allow',  port: 80, proto: 'tcp', log: 'no', from_ip: 'any', to_ip: 'any', direction: 'in', state: 'present', position: 2, comment: 'You can overwrite the default comment'}
}
```

### Execution

Run the playbook:
```bash
ansible-playbook -K -D -i inventory/hosts.yml playbook.yml
```

The ufw-task itself is '[community.general.ufw](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html)'

### Example

**State before:**
```bash
guy@ansible:~$ sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 7424/tcp                   ALLOW IN    Anywhere                   # Ansible managed - confusedService
[ 2] 7429/tcp                   ALLOW IN    Anywhere                   (log) # Ansible managed - nothingImportant
```

**Config**
```yaml
ufw_rules:
  # incoming traffic restrictions
  SecShöl:
    port: 22
    proto: 'tcp'
    log: true
    rule: 'limit'
  RandomWebServer:
    port: 8482
    proto: 'tcp'
  SecureLink:
    port: 54038:54085
    proto: 'udp'
    log: true
    from_ip: '192.168.194.0/28'
  ipsecESP:
    proto: 'esp'
    from_ip: '10.10.10.1'
    to_ip: '10.10.20.254'
  ipsecIKE:
    port: 500,4500
    proto: 'udp'
    from_ip: '10.10.10.1'
    to_ip: '10.10.20.254'
  
  # outgoing traffic restrictions
  denyNtpOutgoing:
    port: 123
    proto: 'udp'
    rule: 'deny'
    direction: 'out'

  # remove those rules:
  confusedService:
    port: 7424
    proto: 'tcp'
    state: 'absent'
  nothingImportant:
    port: 7429
    proto: 'tcp'
    log: true
    present: false
```

**Result:**
```bash
guy@ansible:~$ sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     LIMIT IN    Anywhere                   (log) # Ansible managed - SecShöl
[ 2] 8482/tcp                   ALLOW IN    Anywhere                   # Ansible managed - RandomWebServer
[ 3] 54038:54085/udp            ALLOW IN    192.168.194.0/28           (log) # Ansible managed - SecureLink
[ 4] 10.10.20.254/esp           ALLOW IN    10.10.10.1/esp             # Ansible managed - ipsecESP
[ 5] 10.10.20.254 500,4500/udp  ALLOW IN    10.10.10.1                 # Ansible managed - ipsecIKE
[ 6] 123/udp                    DENY OUT    Anywhere                   (out) # Ansible managed - denyNtpOutgoing
```

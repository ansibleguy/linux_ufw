# Ansible UFW role

Role to configure the ufw service.

**Note:** this role currently only supports debian-based systems

## Functionality

This ansible role will do:
* UFW base-config
* Verify that a ssh-rule is in place
* Rules via using one of two modes
  * The stateful way (*default*)
    * keeps existing rules and adds/removes rules using a rule state
  * The stateless way
    * reset's the ufw state and rules every time
    * after that the new rules get applied

**Note:** Most of this functionality can be opted in or out using the main defaults file and variables!

**Tested:**
* Debian 11

## Requirements

* Community collection: ```ansible-galaxy install -r requirements.yml```


## Usage

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

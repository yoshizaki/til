# inventoryを ini形式とYAML形式で比較する

Cisco Router x 3と Catalyst x 1がいる状態で
Router/Switchを子グループに分割して、それをCisco親グループでまとめています。

```ini
[all:vars]
ansible_ssh_user=ansible
ansible_ssh_pass=password
enable_pass=password

[cisco:children]
router
switch

[router]
R1  ansible_host=192.168.30.171
R2  ansible_host=192.168.30.172
R3  ansible_host=192.168.30.173

[switch]
switch1   ansible_host=192.168.30.170
```


```yaml
---
all:
  vars:
    ansible_ssh_user: ansible
    ansible_ssh_pass: password
    enable_pass: password

cisco:
  children:
    router:
    switch:

router:
  hosts:
    R1:
      ansible_host: 192.168.30.171
    R2:
      ansible_host: 192.168.30.172
    R3:
      ansible_host: 192.168.30.173

switch:
  hosts: 
    switch1:
      ansible_host: 192.168.30.170
```

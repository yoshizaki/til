# ansibleを使用してCisco deviceの情報収集

ansibleは 2.3.2を使用、moduleは ios_factsです。

```shell-session
$ ansible --version
ansible 2.3.2.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
  python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]
```

gather_factsを有効にしていると日付などをansible_xxxから使用できるので trueにしています。
（この例では使ってませんが・・・）
ネットワーク装置なので connectionは localを指定します。

```yaml
---
- hosts: cisco
  gather_facts: true
  connection: local

  tasks:
    - name: cisco device gathering facts.
      ios_facts:
        gather_subset:
          - all
        provider: "{{ cli }}"
      register: result

    - name: output ios_facts
      debug:
        var: result

  vars:
    cli:
      host: "{{ ansible_host }}"
      username: "{{ ansible_ssh_user }}"
      password: "{{ ansible_ssh_pass }}"
      auth_pass: "{{ enable_pass }}"
      transport: cli
```

inventoryもyaml formatで作成しています。
公開鍵認証方式でログインしているので passwordは使用していません。
ユーザも privilege 15にしていますので、enable_passも使用していません。

```yaml
cisco:
  hosts:
    switch1:
      ansible_ssh_host: 192.168.30.170
    R1:
      ansible_ssh_host: 192.168.30.171
    R2:
      ansible_ssh_host: 192.168.30.172
    R3:
      ansible_ssh_host: 192.168.30.173


  vars:
    ansible_ssh_user: ansible
#    ansible_ssh_pass: password
#    enable_pass: password
```


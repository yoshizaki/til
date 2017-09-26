# local_action と task毎のbecome

becomeをタスク毎に分けて実行してみた結果

* local_action で become: false
  * playbook実行ユーザ
* local_action で become: true
  * local root
* remote で become: true
  * remote site root
* remote で become: false
  * ansible_ssh_user

```yaml
---
- name: Testing local_action module
  hosts: all
  gather_facts: false

  pre_tasks:

    - name: localhost id(become false)
      local_action: command id
      changed_when: false
      register: local_result

    - name: output local_result(become false)
      debug:
        var: local_result.stdout

    - name: localhost id(become true)
      local_action: command id
      changed_when: false
      register: local_result
      become: true

    - name: output local_result(become true)
      debug:
        var: local_result.stdout

  tasks:
    - name: remote id(become true)
      command: id
      changed_when: false
      register: remote_result
      become: true

    - name: output remote_result(become true)
      debug:
        var: remote_result.stdout
 
    - name: remote id(become false)
      command: id
      changed_when: false
      register: remote_result
      become: false

    - name: output remote_result(become false)
      debug:
        var: remote_result.stdout
```

実行結果

```
PLAY [Testing local_action module] *****************************************************************************************************************************************************************************************************************************

TASK [localhost id(become false)] ******************************************************************************************************************************************************************************************************************************
ok: [vagrant-machine -> localhost]

TASK [output local_result(become false)] ***********************************************************************************************************************************************************************************************************************
ok: [vagrant-machine] => {
      "local_result.stdout": "uid=1000(tomo) gid=999(docker) groups=999(docker),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lpadmin),124(sambashare),126(libvirtd),128(ubridge)"
}

TASK [localhost id(become true)] *******************************************************************************************************************************************************************************************************************************
ok: [vagrant-machine -> localhost]

TASK [output local_result(become true)] ************************************************************************************************************************************************************************************************************************
ok: [vagrant-machine] => {
      "local_result.stdout": "uid=0(root) gid=0(root) groups=0(root)"
}

TASK [remote id(become true)] **********************************************************************************************************************************************************************************************************************************
ok: [vagrant-machine]

TASK [output remote_result(become true)] ***********************************************************************************************************************************************************************************************************************
ok: [vagrant-machine] => {
      "remote_result.stdout": "uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023"
}

TASK [remote id(become false)] *********************************************************************************************************************************************************************************************************************************
ok: [vagrant-machine]

TASK [output remote_result(become false)] **********************************************************************************************************************************************************************************************************************
ok: [vagrant-machine] => {
      "remote_result.stdout": "uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023"
}

PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
vagrant-machine            : ok=8    changed=0    unreachable=0    failed=0
```





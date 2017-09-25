# [Cisco] show running-config をファイルに保存する

ios_command module でshow running-config を実行して registerで保存した
結果の stdout[0]に改行コード込みの show running-configがあるの copy moduleを使って保存します。

```yaml
---
  - name: save running-config to a file
    hosts: cisco
    gather_facts: false
    connection: local

    tasks:

      - name: show running-config
        ios_command:
          commands:
            - show running-config
          provider: "{{ cli }}"
        register: config_result

      - name: save running-config
        copy:
          content: "{{ config_result.stdout[0] }}" 
          dest: configs/{{ inventory_hostname }}.conf
 ```

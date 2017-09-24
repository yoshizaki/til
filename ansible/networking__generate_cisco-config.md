# [写経] ansibleを使ったCisco装置へのConfig設定

ref.) https://www.youtube.com/watch?v=WXLUgDmvHDI

各装置は事前にlocal machineのpublic-keyを登録済みです。
例では変更を保存(copy run start)していないので、2.2以降ならsave、2.4からsave_when parameterを指定すると保存できます。

# within playbook

ios_config moduleのcommands parameterで投入するConfigを列挙します。

```yaml
---

  - name: Delpoy SNMP configurations
    hosts: cisco
    gather_facts: false
    connection: local

    tasks:

      - name: Deploy SNMP command within playbook
        ios_config:
          commands:
            - snmp-server community public123 RO
            - snmp-server community private123 RW
            - snmp-server location Tokyo/JP
            - snmp-server contact operator@lab.local
          provider: "{{ cli }}"
```

# from config file

configファイルを作成して、それを適用する方法です。

```shell-session
$ cat snmp.cfg
snmp-server community public123 RO
snmp-server community private123 RW
snmp-server location Tokyo/JP
snmp-server contact tomo.yoshizaki@lab.local
```

ホスト指定など途中は省略

```yaml
      - name: Deploy SNMP command from configuration file
        ios_config:
          src: "./snmp.cfg"
          provider: "{{ cli }}"
 ```
 
# from jinja2 template

jinja2 templateを作成して、それを適用する方法です。
jinja2 variablesは group_vars/all.ymlで作成しています。

jinja2 template, RO/RWは複数設定することを想定して forで回しています(例は１つずつ)

```jinja
$ cat templates/snmp.j2
snmp-server location {{ snmp.location }}
snmp-server contact {{ snmp.contact }}
{% for ro_community in snmp.ro %}
snmp-server community {{ ro_community }} RO
{% endfor %}
{% for rw_community in snmp.rw %}
snmp-server community {{ rw_community }} RW
{% endfor %}
```

ホスト指定など途中は省略

```yaml
      - name: Deploy SNMP command from jinja2 template
        ios_config:
          src: "snmp.j2"
          provider: "{{ cli }}"
        tags: snmp-j2
 ```
 
# from auto-generated config file

最後はtemplate-moduleで jinja2 templateからconfigファイルを作成してそのファイルを適用する方法です。
対象装置が多いと何度も config fileを作成しますので run_once:true を指定します。

```yaml
      - name: generate SNMP config file
        template:
          src: "snmp.j2"
          dest: "./configs/snmp-auto.cfg"
        run_once: true
        tags: build

      - name: deploy SNMP from auto-generated file
        ios_config:
          src: "./configs/snmp-auto.cfg"
          provider: "{{ cli }}"
        tags: deploy
```

## cli and jinja2 template variables

providerで使用する cli では公開鍵認証でログインするユーザをprivilege 15に設定しています。
password認証の場合はpasswordなども指定が必要です。

```yaml
$ cat group_vars/all.yml
---
cli:
  host: "{{ ansible_ssh_host }}"
  username: "{{ ansible_ssh_user }}"
  transport: cli

snmp:
  contact: operator@lab.local
  location: Tokyo/JP
  ro:
    - public123
  rw:
    - private123
```



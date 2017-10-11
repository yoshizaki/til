# templateとjinja2をお試し

リスト内には複数のdictがあります。

```yaml
---
a_list:
  - { hostname: client1, param1: 1234, param2: test }
  - { hostname: client2, param1: 5678, param2: validate }
```

playbookでは templateで生成するファイル名に dict内にある hostname の値でファイルを複数作成します。

```yaml
---
- name: more more template/jinja2
  hosts: localhost
  connection: local
  
  vars_files:
    - vars/parameters.yml

  tasks:
    - name: dump list
      debug:
        var: "item"
      with_items:
        - "{{ a_list }}"
        
    - name: generate file
      template:
        src: parameter.j2
        dest: ./parameter_{{ item.hostname }}.cfg
      with_items: "{{ a_list }}"
      run_once: true
```

そのための jinja2 template
listから dictを取り出して、playbookの item.hostnameと比較して
一致したものを templateで作成（データ構造は適当）

```jinja2
{% for dict in dict_in_alist %}
{% if dict.hostname == item.hostname %}
{{ dict.hostname }} : {
  param: {{ dict.param1 }},
  param: {{ dict.param2 }}
  }
{% endif %}
{% endfor %}
```
Faculty: [PIN](https://fict.itmo.ru)

Course: [Network programming](https://itmo-ict-faculty.github.io/network-programming/)

Year: 2024/2025

Group: K3320

Author: Gusev Yaroslav Aleksandrovich

Lab: [Lab2](https://itmo-ict-faculty.github.io/network-programming/education/labs2023_2024/lab2/lab2/)

Date of create: 27.04.2025

Date of finished: 28.04.2025


# Создание второй виртуальной машины с CHR

Была склонирована виртуальная машина с CHR, под новый роутер был создан и настроен клиент OpenVPN.

![image](https://github.com/user-attachments/assets/87c51645-693c-4936-959e-a42ec1acf613)


# Настройка окружения

Ansible был установлен в виртуальное окружение. Были поставлены нужные для работы пакеты:

```ini
ansible==11.5.0
ansible-core==2.18.5
ansible-pylibssh==1.2.2
cffi==1.17.1
cryptography==44.0.2
Jinja2==3.1.6
librouteros==3.4.1
MarkupSafe==3.0.2
packaging==25.0
pycparser==2.22
PyYAML==6.0.2
resolvelib==1.0.1
toml==0.10.2
```

# Написание Ansible Playbook

inventory, хранящий IP-адреса хостов к которым будет подключаться Ansible, параметры подключения и настройки уникальные для хостов (router_id):

```ini
[mikrotik]
10.244.103.2 router_id=1.1.1.1 ansible_user=admin ansible_ssh_pass=1
10.244.103.3 router_id=1.1.1.2 ansible_user=admin ansible_ssh_pass=1

[mikrotik:vars]
ansible_network_os=routeros
ansible_connection=network_cli
```

Конфиг Ansible, указывающий на файл inventory:

```ini
[defaults]
inventory = inventory.ini
```

Файл vars.yaml, содержащий переменные для настройки роутеров:

```yaml
user_config:
  default_group: "read"
  users:
    - username: "other_user"
      password: "2"

ntp_config:
  enabled: "true"
  servers: "servers"
  mode: "unicast"

ospf_config:
  instances:
    - name: "ospf_1"
      router_id: "4"
  areas:
    - name: "backbone"
      area_id: 0.0.0.0
      instance_name: ospf_1
  interface_templates:
    - networks: 0.0.0.0/0
      area: backbone
```

Сам Playbook.yaml, описывающий создание и настройку пользователей, настройку NTP и OSPF:

```yaml
---
- name: Configure routers
  hosts: mikrotik
  gather_facts: no
  module_defaults:
    group/community.routeros.api:
      hostname: "{{ inventory_hostname }}"
      username: "{{ ansible_user }}"
      password: "{{ ansible_ssh_pass }}"
  tasks:
    - include_vars:
        file: vars.yaml
      no_log: true

    - name: Create and modify users
      community.routeros.api_modify:
        path: "user"
        data:
          - name: "{{ user.username }}"
            password: "{{ user.password }}"
            group: "{{ user.group | default(user_config.default_group) }}"
      loop: "{{ user_config.users }}"
      loop_control:
        loop_var: "user"
        label: "{{ user.username }}"
      no_log: true

    - name: Configure NTP Client
      community.routeros.api_modify:
        path: "system ntp client"
        data:
          - enabled: "{{ ntp_config.enabled }}"
            servers: "{{ ntp_config.servers }}"
            mode: "{{ ntp_config.mode }}"

    - name: Create OSPF instances
      community.routeros.api_modify:
        path: "routing ospf instance"
        data:
          - name: "{{ instance.name }}"
            version: "{{ instance.version | default(2) }}"
            router-id: "{{ router_id }}"
      loop: "{{ ospf_config.instances }}"
      loop_control:
        loop_var: "instance"
        label: "{{ instance.name }}"

    - name: Create OSPF areas
      community.routeros.api_modify:
        path: "routing ospf area"
        data:
          - name: "{{ area.name }}"
            area-id: "{{ area.area_id }}"
            instance: "{{ area.instance_name }}"
      loop: "{{ ospf_config.areas }}"
      loop_control:
        loop_var: "area"
        label: "{{ area.name }}"

    - name: Create OSPF interface-templates
      community.routeros.api_modify:
        path: "routing ospf interface-template"
        data:
          - "{{ interface_template }}"
      loop: "{{ ospf_config.interface_templates }}"
      loop_control:
        loop_var: "interface_template"

    - name: gather all facts
      community.routeros.facts:
        gather_subset: all

    - name: gather ospf instance facts
      debug:
        var: ansible_net_ospf_instance

    - name: gather ospf neighbours
      debug:
        var: ansible_net_ospf_neighbor
```

Для работы применялись циклы и использовался идемпотентный модуль api_modify вместо command.

В некоторых местах применялся флаг no_log: true для ограничения вывода в логи чувствительных данных.

Запустим плейбук с помощью команды `ansible-playbook playbook.yaml -v`
Логи настройки:

![image](https://github.com/user-attachments/assets/6cc34ad3-e3b2-44a6-917a-4c177de2b6c9)

Логи вывода конфигурации:

![image](https://github.com/user-attachments/assets/ff62eda5-c574-4461-a875-aad686abee93)



Конфигурация применилась на роутере:

![image](https://github.com/user-attachments/assets/c420eb06-7867-48f6-b900-59a1b4dd1706)

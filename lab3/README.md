University: [ITMO University](https://itmo.ru/ru/)

Faculty: [PIN](https://fict.itmo.ru)

Course: [Network programming](https://itmo-ict-faculty.github.io/network-programming/)

Year: 2024/2025

Group: K3320

Author: Gusev Yaroslav Aleksandrovich

Lab: [Lab3](https://itmo-ict-faculty.github.io/network-programming/education/labs2023_2024/lab3/lab3/)

Date of create: 08.05.2025

Date of finished: 08.05.2025


# Netbox

Был развёрнут Netbox на виртуальной машине в докере с помощью `docker compose`.

![Screenshot_14](https://github.com/user-attachments/assets/e07fe0ff-4368-448a-8532-f494a4bfeb95)

Прокинув порт на локальную машину и перейдя по адресу `localhost:8000` увидим UI Netbox'а.

![image](https://github.com/user-attachments/assets/246b2d22-5d34-4f1c-ae2f-f8c38ee6783d)

Создадим 2 устройства и дадим им IP адреса.

![image](https://github.com/user-attachments/assets/458e19fb-f680-4d0a-909b-2a843bb328f9)

Подключим VM к нашему VPN. Она получает айпи `10.244.103.4`

![Screenshot_16](https://github.com/user-attachments/assets/0a5dc43a-3c87-447e-a162-0dce049b0a84)

# Ansible

## Извлечение конфигурации из Netbox в локальный файл

Был написан плейбук для извлечения информации о устройствах в Netbox и экспорта его в локальный Json файл.

```
---
- name: Get network devices from netbox
  hosts: localhost
  tasks:
    - include_vars:
        file: ../vars.yaml
      no_log: true

    - name: Get netbox devices data
      set_fact:
        devices: "{{ query('netbox.netbox.nb_lookup', 'devices', api_endpoint=netbox.api_address, token=netbox.api_token, validate_certs=false) }}"

    - name: Save data to file
      copy:
        content: "{{ devices | to_json }}"
        dest: devices.json
```

Результат выполнения плейбука:

![Screenshot_17](https://github.com/user-attachments/assets/9eb1b2b0-1858-41ed-824b-6796339c4fbc)

Получившийся json файл приложен к лабораторной работе.

## Конфигурация роутеров

Был написан плейбук для настройки IP адресов и имен роутеров в соответствии с информацией из Netbox.

```
---
- name: Configure routers
  hosts: mikrotik
  gather_facts: no
  module_defaults:
    group/community.routeros.api:
      hostname: "{{ ansible_host }}"
      username: "{{ ansible_user }}"
      password: "{{ ansible_ssh_pass }}"
  vars_files:
    - ../vars.yaml

  pre_tasks:
    - name: Get NetBox devices on localhost
      set_fact:
        all_devices: >-
          {{
            query('netbox.netbox.nb_lookup',
                  'devices',
                  api_endpoint=netbox.api_address,
                  token=netbox.api_token,
                  validate_certs=false)
          }}
      delegate_to: localhost
      run_once: true

    - name: Select device for this host
      set_fact:
        device: >-
          {{
            all_devices
            | selectattr('value.name', 'equalto', inventory_hostname)
            | list
            | first
          }}
      delegate_to: localhost

    - debug:
        msg:
          - "router name: {{ device.value.name }}"
          - "primary ipv4: {{ device.value.primary_ip.address }}"

  tasks:
    - name: Set system identity
      community.routeros.api_modify:
        path: "system identity"
        data:
          - name: "{{ device.value.name }}"

    - name: Set IP Address
      community.routeros.api_modify:
        path: "ip address"
        data:
          - interface: "ovpn-out1"
            address: "{{ device.value.primary_ip.address }}"
```

Маппинг роутера к устройству Netbox был сделан по имени хоста в инвентаре Ansible.

```
[mikrotik]
router-1 ansible_host=10.244.103.2 router_id=1.1.1.1 ansible_user=admin ansible_ssh_pass=1
router-2 ansible_host=10.244.103.3 router_id=1.1.1.2 ansible_user=admin ansible_ssh_pass=1
```

![image](https://github.com/user-attachments/assets/7466f9c6-32c1-443d-a9dc-0e7c362fb155)

Конфигурация успешно применилась:

![Screenshot_18](https://github.com/user-attachments/assets/8f2be234-3c89-420d-9b99-3fafe14e1bca)

![Screenshot_19](https://github.com/user-attachments/assets/332e75ce-b7bc-419d-ab3a-7f52579d2d37)

# Сбор серийных номеров с роутеров и добавление их в Netbox

Был написан и запущен плейбук:

```
---
- name: Configure routers
  hosts: mikrotik
  gather_facts: no
  module_defaults:
    group/community.routeros.api:
      hostname: "{{ ansible_host }}"
      username: "{{ ansible_user }}"
      password: "{{ ansible_ssh_pass }}"
    netbox_device:
      netbox_url: "{{ netbox.api_address }}"
      netbox_token: "{{ netbox.api_token }}"
      validate_certs: false
  vars_files:
    - ../vars.yaml

  tasks:
    - name: Get serial number
      community.routeros.command:
        commands:
          - /system license print
      register: serial_number_info

    - name: Print serial number
      debug:
        msg: "{{ serial_number_info.stdout_lines[0][0].split(' ')[1] }}"

    - name: Update device serial number in NetBox
      delegate_to: localhost
      netbox.netbox.netbox_device:
        data:
          name: "{{ inventory_hostname }}"
          serial: "{{ serial_number_info.stdout_lines[0][0].split(' ')[1]}}"
        state: present
```

![Screenshot_21](https://github.com/user-attachments/assets/dfa112e4-3add-47c2-9ec5-e61f2e013a54)

Результат успешный:
![Screenshot_22](https://github.com/user-attachments/assets/7b1d47d8-1069-4787-a45a-cbb73f8e76b8)

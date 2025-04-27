![image](https://github.com/user-attachments/assets/d5598d00-0f61-4042-b07c-7498c1ce3a85)University: [ITMO University](https://itmo.ru/ru/)

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

```
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

inventory, хранящий хосты к которым будет подключаться Ansible:

```yaml
[mikrotik]
10.244.103.2 mikrotik_user=admin mikrotik_pass=1 router_id=1.1.1.1 ansible_user=admin ansible_ssh_pass=1
10.244.103.3 mikrotik_user=admin mikrotik_pass=1 router_id=1.1.1.2 ansible_user=admin ansible_ssh_pass=1

[mikrotik:vars]
ansible_network_os=routeros
ansible_connection=network_cli
```




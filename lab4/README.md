
University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K3320

Author: Gusev Yaroslav

Date of create: 25.06.2025

Date of finished: 25.06.2025

# Лабораторная работа 4

Установим на виртуальной машине Ubuntu Server и поставим необходимые зависимости.

![image](https://github.com/user-attachments/assets/7ed062c6-d871-4237-9569-4f67a02a43f4)


## Implementing Basic Forwarding

В первой части задания реализуем базовую переадресацию для ipv4 согласно заданию Implementing Basic Forwarding. После компиляции P4-программы и настройки таблиц маршрутизации все узлы обменялись сообщениями.

Схема:

![image](https://github.com/user-attachments/assets/ebed908b-34a7-4533-a3ee-54c10a9941f6)


![Screenshot_6](https://github.com/user-attachments/assets/69d00b86-6d9e-414b-b06c-72c9c1c08571)



## Implementing Basic Tunneling
Во второй части работы передаем пакеты через виртуальный туннель.
Раньше пакеты шли напрямую. Теперь добавили специальный заголовок myTunnel_t.

Схема:

![image](https://github.com/user-attachments/assets/d67b0b2d-5fb3-4ab0-9fc3-370ca5ed5c9c)


Поднимаем узлы

![Screenshot_7](https://github.com/user-attachments/assets/57accb43-6f86-46a8-9127-f7522f8cbe1a)

Проверяем связность, отправляя с ноды h1 на h2 сначала сообщение без туннеля, потом через туннель.

Логи h1:

Без туннеля:

![image](https://github.com/user-attachments/assets/b6b9c0c3-364a-4f90-af47-60058b44d10b)

Через туннель:

![image](https://github.com/user-attachments/assets/d0193ae9-b9a9-4ae8-87d5-925bb705cf80)


Логи Принимающей стороны (h2):

![image](https://github.com/user-attachments/assets/bf4a42ed-55af-4b31-91aa-8c6ccd921bc5)

В первом случае видим, что отсутствует наш заголовок myTunnel_t


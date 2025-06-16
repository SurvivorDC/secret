
Оглавление
1)	Настройте имена устройств согласно топологии. Используйте полное доменное имя	2
2)	На всех устройствах необходимо сконфигурировать IPv4	2
3)	Создание локальных учетных записей	2
4)	Настройка безопасного удаленного доступа на серверах HQ-SRV и BRSRV:	3
5)	Настройка протокола динамической конфигурации хостов.	3
6)	Между офисами HQ и BR необходимо сконфигурировать ip туннель	4
7)	Обеспечьте динамическую маршрутизацию	6
8)	Настройка Samba AD-DC	10




1)	Настройте имена устройств согласно топологии. Используйте полное доменное имя

hostnamectl set-hostname «имя_машины»; exec bash

2)	На всех устройствах необходимо сконфигурировать IPv4

Для устройств с графическим интерфейсом ПРАВОЙ КЛАВИШЕЙ ПО СЕТИ, настроить ipv4 и НЕ ЗАБЫТЬ СОХРАНИТЬ

для устройств БЕЗ графического интерфейса пользуемся nmtui

@@@@добавить скрин@@@@

![Снимок](https://github.com/user-attachments/assets/8b6a1ee9-81a6-4733-bae3-8c9134e9815f)

3)	Создание локальных учетных записей

useradd -m -u 1010 sshuser

passwd sshuser

nano /etc/sudoers

sshuser ALL=(ALL:ALL)NOPASSWD:ALL

ctrl+x

y

enter
 
4)	Настройка безопасного удаленного доступа на серверах HQ-SRV и BR-SRV:

на серверах 

nano /etc/mybanner

В этом НОВОМ ПУСТОМ файле пишем Authorized access only

ctrl+x

y

enter

nano /etc/openssh/sshd_config

находим строчки
 
#port 22, раскоменчиваем и пишем port 2024

Banner /etc/mybanner

MaxAuthTries 2

ДОБАВИТЬ строчку

AllowUsers sshuser

Ctrl+x

Y

enter

systemctl restart sshd.service

5)	Настройка протокола динамической конфигурации хостов.

Настройки проводим на HQ-RTR

nano /etc/sysconfig/dhcpd

DHCPARGS=ens35

ctrl+x

y

enter

cp /etc/dhcp/dhcpd.conf{.example,}

nano /etc/dhcp/dhcpd.conf

В этом файле должны быть строки

option domain-name “au-team.irpo”;

option domain-name-servers 172.16.0.2;


default-lease-time 6000;

max-lease-time 72000;


authoritative;

subnet 172.16.0.0 netmask 255.255.255.192 {
	
 range 172.16.0.3 172.16.0.8;
	
 option routers 172.16.0.1;

}

ctrl+x

y

enter

systemctl enable –now dhcpd

6)	Между офисами HQ и BR необходимо сконфигурировать ip туннель

Перед настройкой самого тоннеля необходимо убедиться, что на ISP включён forwarding IPv4

на ISP

nano /etc/net/sysctl.conf

и меняем строчку net ipv4 forwarding значение на 1

![Снимок1](https://github.com/user-attachments/assets/52cf4dd8-9e9d-4415-87b5-7c46fd57f23a)

 
Настраиваем GRE через nmtui

BR-RTR


 ![Снимок 2](https://github.com/user-attachments/assets/ea521843-e4b9-4103-8023-3a20e6d65753)

 
HQ-RTR


![Снимок 3](https://github.com/user-attachments/assets/473a0355-b2a0-4741-9fbb-d1f5ef002586)
 
7)	Обеспечьте динамическую маршрутизацию
Настраиваем OSPF

![Снимок 4](https://github.com/user-attachments/assets/9635e356-2fa5-41d1-84cb-b02f819996c5)


HQ-RTR

![Снимок 5](https://github.com/user-attachments/assets/6efc288f-4a7e-4b8e-a534-96754485c930)


![Снимок 6](https://github.com/user-attachments/assets/4dfae681-5a5d-406f-a14c-a1f9b17e46be)


systemctl restart frr

BR-RTR
Повторяем со своими адресами


Настройка DNS для офисов HQ и BR.

На HQ-SRV

nano /etc/bind/options.conf

Меняем выделенные строчки
 

![Снимок 7](https://github.com/user-attachments/assets/02db277f-d3ef-4f4c-9a1d-ca7bba9f1ea1)


Systemctl enable bind —now
 
Nano /etc/bind/local.conf

![Снимок 8](https://github.com/user-attachments/assets/209286c2-eb09-41f0-8e19-06b536f69290)

 

CD /etc/bind/zone
CP localdomain au.db
CP 127.in-addr.arpa 0.db
Chown root:named {au,0}.db
 
Nano au.db

![Снимок 9](https://github.com/user-attachments/assets/e6fde1e9-a5b8-4b44-8e4c-35ad0c0ce2dc)


 
Nano 0.db

![Снимок 10](https://github.com/user-attachments/assets/96246847-29a4-4d17-8da4-ffd573e8e64b)


 
Systemctl restart bind 

Проверка host hq-rtr.au-team.irpo

Должен выдать IP

8)	Настройка Samba AD-DC

HQ-SRV

Произведём временное отключение интерфейсов. Обязательно перед началом настройки samba!

nmtui

![11](https://github.com/user-attachments/assets/13c8c0b9-a9ce-42ac-9d58-93e332649e00)


 
grep -q 'bind-dns' /etc/bind/named.conf || echo 'include "/var/lib/samba/bind-dns/named.conf";' >> /etc/bind/named.conf

nano /etc/bind/options.conf


 ![12](https://github.com/user-attachments/assets/cbcd6e06-90c8-48ae-8acc-3cb824cecb9c)


![13](https://github.com/user-attachments/assets/742f8705-778c-4200-a3ba-2d1ab30685cf)



systemctl stop bind

nano /etc/sysconfig/network


![14](https://github.com/user-attachments/assets/cd38126f-78e3-456f-a931-0c712258d396)





 
domainname au-team.irpo

rm -f /etc/samba/smb.conf

rm -rf /var/lib/samba

rm -rf /var/cache/samba

mkdir -p /var/lib/samba/sysvol

samba-tool domain provision


![15](https://github.com/user-attachments/assets/b18c86a9-ad88-4b1e-9235-0f2812ba44f6)


 
systemctl enable --now samba
systemctl enable --now bind #если бинд не запускается то делаем следующие шаги:
1.	nano /etc/bind/named.conf

![16](https://github.com/user-attachments/assets/3b3e10d7-6369-4b59-b080-6c90bf945b58)


 
2.	systemctl restart bind
3.	проверяем что все работает командой systemctl status bind	 



![17](https://github.com/user-attachments/assets/04df0a8a-7c0f-490e-9e75-35ebfc0eaa24)



nano /etc/krb5.conf

![18](https://github.com/user-attachments/assets/e90baf89-3c88-4731-8a81-e9c904ea64e3)

 

samba-tool domain info 127.0.0.1

![19](https://github.com/user-attachments/assets/81d85224-b5e9-4d2e-b683-8fce19688120)


 
kinit administrator@au-team.irpo
 
Создайте 5 пользователей для офиса HQ:
Прописываем команду admc
В открывшимся окне разворачиваем au-team.irpo
Открываем вкладку users и создаем пользователей
имена пользователей формата user№.hq
HQ-CLI
Указываем DNS сервер домена
 
 
 ![20](https://github.com/user-attachments/assets/edf48a9b-7863-4b46-ad7f-d1c1bf42eaea)

 

![21](https://github.com/user-attachments/assets/34d026d5-fb11-4b7b-88bb-0e5702a4e183)



![22](https://github.com/user-attachments/assets/701f5696-e138-49f4-a0d3-d79491cca1a4)




![23](https://github.com/user-attachments/assets/1a9d1838-b0ff-4402-9f71-7e9723396bcb)



![24](https://github.com/user-attachments/assets/62ea032e-1d90-49b1-aa2a-b3eb077ef53c)


 
 
После этого перезагружаем систему командой reboot и пробуем войти под учетной записью administrator@au-team.irpo


![image](https://github.com/user-attachments/assets/fc7fb17c-e107-40ce-bfdf-be9fbd2689fe)
![image](https://github.com/user-attachments/assets/fc7fb17c-e107-40ce-bfdf-be9fbd2689fe)

![image](https://github.com/user-attachments/assets/4613a2f3-91c0-4820-9e22-305cd5d07934)
![image](https://github.com/user-attachments/assets/4613a2f3-91c0-4820-9e22-305cd5d07934)








![image](https://github.com/user-attachments/assets/4e19c52d-3a58-495c-8b54-f3865f142f85)
![image](https://github.com/user-attachments/assets/4e19c52d-3a58-495c-8b54-f3865f142f85)


![image](https://github.com/user-attachments/assets/a407afe7-371d-4e3e-b438-90bd289f5159)
![image](https://github.com/user-attachments/assets/a407afe7-371d-4e3e-b438-90bd289f5159)


![image](https://github.com/user-attachments/assets/c3cc10a5-8ac1-496f-9fc2-643d5e6685f6)
![image](https://github.com/user-attachments/assets/c3cc10a5-8ac1-496f-9fc2-643d5e6685f6)


![image](https://github.com/user-attachments/assets/f999966c-d471-45c3-8a06-28d73acce391)
![image](https://github.com/user-attachments/assets/f999966c-d471-45c3-8a06-28d73acce391)


![image](https://github.com/user-attachments/assets/b2d0e3ec-47c4-4df5-a691-b5c2b5dd037c)
![image](https://github.com/user-attachments/assets/b2d0e3ec-47c4-4df5-a691-b5c2b5dd037c)

![image](https://github.com/user-attachments/assets/43e1e373-f463-407d-8e97-14c86456f74b)
![image](https://github.com/user-attachments/assets/43e1e373-f463-407d-8e97-14c86456f74b)

![image](https://github.com/user-attachments/assets/56813ac5-7ae6-49a3-8fcd-4d219cd99444)
![image](https://github.com/user-attachments/assets/56813ac5-7ae6-49a3-8fcd-4d219cd99444)

![image](https://github.com/user-attachments/assets/f6f63bb6-c1d0-4f4b-b468-dfe95d43f691)
![image](https://github.com/user-attachments/assets/f6f63bb6-c1d0-4f4b-b468-dfe95d43f691)


![image](https://github.com/user-attachments/assets/c5f899d6-e5b5-4420-bc06-45e53dd324d1)
![image](https://github.com/user-attachments/assets/c5f899d6-e5b5-4420-bc06-45e53dd324d1)


![image](https://github.com/user-attachments/assets/e8a21a75-c092-4418-b702-06acf36b68ce)
![image](https://github.com/user-attachments/assets/e8a21a75-c092-4418-b702-06acf36b68ce)


![image](https://github.com/user-attachments/assets/dcc46e25-07f3-4fc2-8bac-cd5be1bff883)
![image](https://github.com/user-attachments/assets/dcc46e25-07f3-4fc2-8bac-cd5be1bff883)


![image](https://github.com/user-attachments/assets/a4fc42ec-955b-4da3-8366-f339dfb43981)
![image](https://github.com/user-attachments/assets/a4fc42ec-955b-4da3-8366-f339dfb43981)


![image](https://github.com/user-attachments/assets/052f02ca-ec9d-4fca-a2e6-b6b38f447872)
![image](https://github.com/user-attachments/assets/052f02ca-ec9d-4fca-a2e6-b6b38f447872)


![image](https://github.com/user-attachments/assets/76393245-3791-4a3f-bf26-a2e97c66c044)
![image](https://github.com/user-attachments/assets/76393245-3791-4a3f-bf26-a2e97c66c044)










































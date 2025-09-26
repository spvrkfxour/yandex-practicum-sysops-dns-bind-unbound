## Задание №1
Создать две виртуальные машины: первая выступит в качестве DNS-сервера на Bind, а на второй будет веб-сайт

## Решение задания №1
На VM-01 устанавливаем пакет bind9
```console
sudo apt update && sudo apt upgrade -y
sudo apt install bind9 -y
```

В конфигурации named.conf.options указываем директорию где будет храниться кэш и другие временные файлы, ip адреса DNS серверов к которым будут идти запросы если нет записи в локальном DNS и проверку подписи DNSSEC

```console
sudo nano /etc/bind/named.conf.options
```
<img width="614" height="367" alt="image" src="https://github.com/user-attachments/assets/d8cbb78a-a596-418c-8898-faaf8a579f6c" />
<br><br>

Настраиваем локальную доменную зону в named.conf.local

```console
sudo nano /etc/bind/named.conf.local
```

<img width="620" height="233" alt="image" src="https://github.com/user-attachments/assets/10726640-a365-4d6e-a404-4dc2403a485d" />
<br><br>

И зону обратного просмотра

<img width="313" height="89" alt="image" src="https://github.com/user-attachments/assets/eb1b946c-d60d-4d54-ac81-93bd76fc67d7" />
<br><br>

Добавим записи для каждой зоны в db.s758434.local и db.10. Используем стандартную конфигурацию

```console
sudo cp /etc/bind/db.local /etc/bind/db.s758434.local
sudo cp /etc/bind/db.127 /etc/bind/db.10
sudo nano /etc/bind/db.s758434.local
```

<img width="620" height="240" alt="image" src="https://github.com/user-attachments/assets/091ed5fa-df38-4ded-af4a-172dba3b4c9f" />
<br><br>

```console
sudo nano /etc/bind/db.10
```

<img width="618" height="231" alt="image" src="https://github.com/user-attachments/assets/23549e1e-be2a-4edd-9a80-904df240d7b0" />
<br><br>

Проверим конфиг и зоны
```console
sudo named-checkconf
sudo named-checkzone s758434.local /etc/bind/db.s758434.local
sudo named-checkzone 10.in-addr.arpa /etc/bind/db.10
```

Перезапускаем bind
```console
sudo systemctl restart bind9
sudo systemctl status bind9
```

Проверим

<img width="604" height="184" alt="image" src="https://github.com/user-attachments/assets/afb0d606-9559-49ac-8d70-b8f1b2c8115b" />
<br><br>

На VM-02 разворачиваем сервер
```console
sudo apt update && sudo apt upgrade -y
python3 -m http.server
```

<img width="467" height="333" alt="image" src="https://github.com/user-attachments/assets/949c63f5-a2bf-42a1-a985-9b9531c7ed4a" />
<br><br>

В качестве DNS сервера указываем VM-01

<img width="502" height="379" alt="image" src="https://github.com/user-attachments/assets/5eda06b0-b5d2-4fa7-88ae-43d00c2a8133" />
<br><br>

Проверим

<img width="323" height="127" alt="image" src="https://github.com/user-attachments/assets/585d5295-e728-4246-afb1-b596fd9cf47e" />
<br><br>

<img width="372" height="128" alt="image" src="https://github.com/user-attachments/assets/0931bfda-eb28-4dc4-934c-3e7b607deb0f" />
<br><br>

## Задание №2
Настроить кэширующий DNS-сервер на базе Unbound

## Решение задания №2
На VM-01(DNS сервер) устанавливаем пакет unbound
```console
sudo apt update && sudo apt upgrade -y
sudo apt install unbound -y
```

Создаем конфигурационный файл
```console
sudo rm /etc/unbound/unbound.conf.d/*
sudo nano /etc/unbound/unbound.conf.d/main.conf
```

Разрешаем запросы с подсетей VM-01 и VM-02 и запрещаем остальные. Сервис будет слушать все интерфейсы. Записи в кэше будут жить два часа

<img width="424" height="518" alt="image" src="https://github.com/user-attachments/assets/e1eda32f-292e-4b18-9265-d22c792b85e2" />
<br><br>

Проверим синтаксис конфигурации
```console
sudo unbound-checkconf /etc/unbound/unbound.conf.d/main.conf
```

Проверим свободен ли порт 53
```console
sudo ss -tulpn | grep :53
```

Порт занят службой systemd-resolved и bind. Добавим в файл /etc/systemd/resolved.conf строку DNSStubListener=no чтобы служба перестала слушать порт и перезапустим её
```console
sudo systemctl restart systemd-resolved.service
```

Для bind изменим порт на 5353. Bind будет отдавать DNS только unbound на этом же сервере

<img width="374" height="41" alt="image" src="https://github.com/user-attachments/assets/fe9c66b3-98da-4640-952a-f0d6a59b7440" />
<br><br>

Перезапустим bind и unbound
```console
sudo systemctl restart bind9
sudo systemctl restart unbound
```

Проверим порты 53 и 5353 для unbound и bind соответственно
```console
sudo ss -tulpn | grep :53
```

<img width="618" height="162" alt="image" src="https://github.com/user-attachments/assets/25756dab-6065-4cde-9065-1f87960c123d" />
<br><br>

Проверим
```console
 sudo systemctl status unbound
```

<img width="617" height="166" alt="image" src="https://github.com/user-attachments/assets/c6d70465-383c-4ad8-81c2-cc450137436a" />
<br><br>

На VM-02 выполним запрос с DNS сервера Google
```console
dig @8.8.8.8 practicum.yandex.ru
```

<img width="619" height="339" alt="image" src="https://github.com/user-attachments/assets/e8586c8f-2b51-486b-9bb4-4c91e522a2a1" />
<br><br>

Время выполнения запроса - 23 миллисекунды

На VM-02 выполним запрос с unbound сервера
```console
dig @10.10.0.206 practicum.yandex.ru
```

<img width="617" height="321" alt="image" src="https://github.com/user-attachments/assets/a9e3718e-1a60-4d43-bb4f-d44dc219a354" />
<br><br>

Время выполнения первого запроса - 151 миллисекунда

Повторим запрос. Время запроса должно уменьшиться

<img width="619" height="327" alt="image" src="https://github.com/user-attachments/assets/50af0a6a-c335-4567-a47e-43aa8d41f8c9" />
<br><br>

Время выполнения повторного запроса - 3 миллисекунды

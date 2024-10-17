Server setup

Ufw config

Install
apt install ufw

Enable and config
ufw default deny incoming && ufw default allow outgoing
ufw allow 22
ufw allow 222
other ports

Apply config
ufw disable && ufw enable

ssh config

add user
sudo adduser www
sudo adduser www sudo

generate ssh key:
ssh-keygen -t ed25519

generate ssh pub key:
cat ~/.ssh/id_ed25519.pub | pbcopy

edit ssh config:
sudo nano /etc/ssh/sshd_config

copy ssh key on server:
ssh-copy-id -i .ssh/_srv_ www@__

edit ssh config settings:
AllowUsers www

PermitRootLogin no

PasswordAuthentication no

Port 222

Banner /etc/banner

restart ssh server:
sudo service ssh restart

Port knocking

## Установка knockd:
```console
sudo apt install -y knockd
```

## Редактирование его настроек:
```console
sudo nano /etc/knockd.conf
```

### Настройки:

[options]

    UseSyslog

    Interface = enp3s0

[SSH]

    sequence    = 7000,8000,9000

    seq_timeout = 5

    tcpflags    = syn

    start_command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 45916 -j ACCEPT

    stop_command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 45916 -j ACCEPT

    cmd_timeout   = 60

Здесь eth0 это сетевой интерфейс, проверяется командой:
```console
ip a
```

## Автозапуск:
```console
sudo vim /etc/default/knockd
```
    START_KNOCKD=1

    KNOCKD_OPTS="-i eth0"


Сетевой интерфейс аналогично подставляется актуальный вместо eth0.

## Старт knockd:
```console
sudo systemctl start knockd
sudo systemctl enable knockd
sudo systemctl status knockd
```


## Настройка iptables и перманентное сохранение этих настроек:
```console
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 45916 -j REJECT
sudo apt install iptables-persistent
sudo service netfilter-persistent save
```

## Просмотр настроек:
```console
sudo iptables -L --line-numbers
```

## Постучаться в три порта — 
7000, 8000 и 9000 сервера с IP 46.148.229.113:
for x in 7000 8000 9000; do nmap -Pn --max-retries 0 -p $x 46.148.229.113; done 

## Сброс всех настроек iptables:
```console
sudo iptables -F
sudo iptables -L --line-numbers
```

## Запрет ping хоста:
```console
sudo iptables -A INPUT -p icmp --icmp-type 8 -j DROP
```

## Удаление этого правила:
```console
sudo iptables -D INPUT -p icmp --icmp-type 8 -j DROP
```

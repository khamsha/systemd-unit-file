# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.vm.box = "centos/stream8"
  config.vm.synced_folder ".", "/vagrant"

  config.vm.provision "shell", inline: <<-SHELL
    sudo -i
    # Создаем файл с конфигурацией для нашего сервиса
    cat << 'EOF' > /etc/sysconfig/watchlog
# Configuration file for my watchlog service
# Place it to /etc/sysconfig

# File and word in that file that we will be monitoring
WORD="ALERT"
LOG=/var/log/watchlog.log
EOF

    # Создаем файл с логами
    cat << 'EOF' > /var/log/watchlog.log
Some text
And some more text
And some other logs
ALERT
EOF

    # Содаем скрипт сервиса
    cat << 'EOF' > /opt/watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`
    
if grep $WORD $LOG &> /dev/null
then
  logger "$DATE: I found word, Master!"
else
  exit 0
fi
EOF

    # Добавляем права на запуск файла со скриптом
    chmod +x /opt/watchlog.sh
    
    # Создаем юнит для сервиса
    cat << 'EOF' > /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service
    
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
EOF

    # Создаем юнит для таймера
    cat << 'EOF' > /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second
    
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
EOF

    # Запускаем таймер и сервис
    systemctl start watchlog.timer
    systemctl start watchlog.service

    # Устанавливаем spawn-fcgi и необходимые для него пакеты
    yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y

    # Переписываем файл /etc/sysconfig/spawn-fcgi
    cat << 'EOF' > /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
EOF

    # Создаем юнит для сервиса spawn-fcgi
    cat << 'EOF' > /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service
After=network.target
 
[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

    # Запускаем spawn-fcgi
    systemctl start spawn-fcgi

    # Дополняем юнит apache httpd возможностью запустить несколько инстансов сервера с разными конфигами (через параметр параметр %I)
    cp /usr/lib/systemd/system/httpd.service /etc/systemd/system/httpd.service
    sed -i '/^Environment.*/a EnvironmentFile=/etc/sysconfig/httpd-%I' /etc/systemd/system/httpd.service

    # Создаем файлы окружения и задаем опцию для запуска веб-сервера с необходимым конфиг файлом
    cat << EOF > /etc/sysconfig/httpd-first
OPTIONS=-f conf/first.conf
EOF
    
    cat << EOF > /etc/sysconfig/httpd-second
OPTIONS=-f conf/second.conf
EOF

    # Настраиваем конфиг файлы, чтобы у них были уникальные опции Listen и PidFile
    cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
    cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf

    sed -i '/^ServerRoot.*/a PidFile "/var/run/httpd-first.pid"' /etc/httpd/conf/first.conf
    sed -i '/^ServerRoot.*/a PidFile "/var/run/httpd-second.pid"' /etc/httpd/conf/second.conf

    sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/second.conf

    # Запускаем веб-серверы
    systemctl start httpd@first
    systemctl start httpd@second

  SHELL
end

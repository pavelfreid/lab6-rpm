# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/stream8"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provision "shell", inline: <<-SHELL
    sudo -i
    # Последовательно устанавливаем необходимые пакеты. Установка пакетов одной командой выдает ошибку
    yum install -y redhat-lsb-core
    yum install -y wget
    yum install -y rpmdevtools
    yum install -y rpm-build
    yum install -y createrepo
    yum install -y yum-utils
    yum install -y gcc
    
    # Чтобы создать собственный пакет Nginx с поддержкой openssl, загрузим и установим исходники пакета
    wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
    rpm -i nginx-1.20.2-1.el8.ngx.src.rpm
    wget https://github.com/openssl/openssl/archive/refs/tags/OpenSSL_1_1_1s.tar.gz
    tar -xvf OpenSSL_1_1_1s.tar.gz
    # С помощью команды rpm -i каталог rpmbuild был установлен в корневом домашнем каталоге 
    # Устанавливаем необходимые зависимости
    yum-builddep -y /root/rpmbuild/SPECS/nginx.spec
    # Изменение файла спецификации для включения опции openssl, пересборка пакета rpm и его установка
    sed -i 's#--with-debug#--with-openssl=/home/vagrant/openssl-OpenSSL_1_1_1s#' /root/rpmbuild/SPECS/nginx.spec
    rpmbuild -bb /root/rpmbuild/SPECS/nginx.spec
    yum localinstall -y /root/rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm
    systemctl start nginx
    # Создаем репозиторий пакетов, включаем туда новый образ nginx и пакеты percona
    mkdir /usr/share/nginx/html/repo
    cp /root/rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm /usr/share/nginx/html/repo/
    wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
    cp percona-orchestrator-3.2.6-2.el8.x86_64.rpm /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
    createrepo /usr/share/nginx/html/repo/
    # Настройка nginx
    sed -i '/index.html index.htm;/a\        autoindex on;' /etc/nginx/conf.d/default.conf
    # Проверка синтаксиса и перезагрузка nginx
    nginx -t
    nginx -s reload
    # Добавляем репозиторй /etc/yum.repos.d и проверяем установку
    cat <<-EOF > /etc/yum.repos.d/otus.repo
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
    yum install percona-orchestrator.x86_64 -y
  SHELL
end

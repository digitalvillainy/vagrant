# -*- mode: ruby -*-
# vi: set ft=ruby :

# neoan3 vagrant box
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/bionic64"

  config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.synced_folder ".", "/var/www/html"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "neoan3-app"
  end

  config.vm.provision "shell", inline: <<-SHELL
    # Install apache server , mysql
    echo "** 1/6 Install Apache & MySQL **"
    apt-get install software-properties-common -y
    add-apt-repository ppa:ondrej/php -y
    apt-get update
    apt-get install -y apache2
    apt-get install -y mysql-server

    # Create default database
    echo "** 2/6 Create default database `neoan3` **"
    mysql -e \"create database neoan3\"

    # Install PHP8 & modules
    echo "** 3/6 Install PHP8 & modules **"
    apt-get install -y php8.0 libapache2-mod-php8.0 php8.0-{mysql,zip,xml,curl,mbstring} curl git

    # Install & setup Composer
    echo "** 4/6 Install & Setup composer**"
    curl -sS https://getcomposer.org/installer -o composer-setup.php
    php composer-setup.php --install-dir=/usr/local/bin --filename=composer

    # Install & setup neoan3 cli
    echo "** 5/6 Install & Setup neoan3 cli**"
    sudo -u vagrant -i composer global require neoan3/neoan3
    mkdir /credentials
    echo '{"testing_db":{"host":"localhost","user":"neoan3","name":"neoan3","assume_uuids":true}}' > /credentials/credentials.json
    chown vagrant:vagrant -R /credentials
    echo "PATH=$PATH:/home/vagrant/.config/composer/vendor/bin" >> /home/vagrant/.profile
    echo "cd /var/www/html" >> /home/vagrant/.profile

    # Configure Apache
    echo "** 6/6 configure Apache**"
    echo "<VirtualHost *:80>
        DocumentRoot /var/www/html
        AllowEncodedSlashes On
        <Directory /var/www/html>
            Options +Indexes +FollowSymLinks
            DirectoryIndex index.php index.html
            Order allow,deny
            Allow from all
            AllowOverride All
            <IfModule php_module>
                    AddHandler application/x-httpd-php .php
            </IfModule>
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>" > /etc/apache2/sites-available/000-default.conf
    a2enmod rewrite
    service apache2 restart
    echo "** neoan3 box running... visit http://192.168.33.10 in your browser for to view the application **"
    echo "** RUN: `vagrant ssh` **"
    echo "** ... synced working directory .. **"
  SHELL
end

$script = <<SCRIPT
apt-get -y install git
	chown -R vagrant:vagrant /var/log/apache2
	cat << EOF > /etc/apache2/sites-available/vagrant_vhost.conf
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /vagrant
        LogLevel debug

        ErrorLog /var/log/apache2/error.log
        CustomLog /var/log/apache2/access.log combined

        <Directory /vagrant>
            AllowOverride All
            Require all granted
        </Directory>
</VirtualHost>
EOF
  a2dissite 000-default
	a2ensite vagrant_vhost

	a2enmod rewrite

	update-rc.d apache2 enable
	add-apt-repository -y ppa:ondrej/php5-5.6
	apt-get update

	apt-get -y install nodejs npm

  apt-get -y install php5 php5-curl php5-mysql php5-sqlite php5-xdebug php5-mcrypt php5-gd php5-intl php5-xsl

	php5enmod mcrypt
	php5enmod gd
	php5enmod intl
	php5enmod xsl

	service apache2 restart

	curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

	debconf-set-selections <<< 'mysql-server mysql-server/root_password password test'
	debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password test'

  apt-get -y install mysql-client-5.6 mysql-server-5.6

  echo "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION" | mysql -u root --password=test
	echo "GRANT PROXY ON ''@'' TO 'root'@'%' WITH GRANT OPTION" | mysql -u root --password=test

  service mysql restart

	echo "phpmyadmin phpmyadmin/dbconfig-install boolean true" | debconf-set-selections
	echo "phpmyadmin phpmyadmin/app-password-confirm password test" | debconf-set-selections
	echo "phpmyadmin phpmyadmin/mysql/admin-pass password test" | debconf-set-selections
	echo "phpmyadmin phpmyadmin/mysql/app-pass password test" | debconf-set-selections
	echo "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2" | debconf-set-selections
	apt-get -y install phpmyadmin > /dev/null 2>&1

  apt-get install -y build-essential libsqlite3-dev ruby1.9.1-dev
	gem install mime-types -v 2.6.2
	gem install mailcatcher
	mailcatcher --http-ip=0.0.0.0

	echo "sendmail_path = /usr/bin/env $(which catchmail) -f test@local.dev" | sudo tee /etc/php5/mods-available/mailcatcher.ini
	sudo php5enmod mailcatcher

  service apache2 restart

SCRIPT

Vagrant.configure("2") do |config|
	config.vm.box = "ubuntu/trusty64"
  config.vm.network "forwarded_port", guest: 80, host: 8080
	config.vm.network "forwarded_port", guest: 1080, host: 1080
	config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", :owner => "www-data", :group => "www-data", :mount_options => ["dmode=777","fmode=666"]
	config.vm.provider :virtualbox do |vb|
    vb.customize [
      "modifyvm", :id,
      "--cpuexecutioncap", "50",
      "--memory", "2048",
    ]
    config.vm.provision "shell", inline: $script
  end
end

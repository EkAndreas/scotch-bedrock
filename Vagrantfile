# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    ### change these two parameters unique to your project!
    #================================================================

    hostname = "scotchbox"
    ip = "192.168.33.10"

    #================================================================

    path_apache_www = "/var/www"
    path_apache_conf = "/etc/apache2/sites-available"
    name_realhost = "#{hostname}.dev"
    name_conf = "#{name_realhost}.conf"
    name_docroot = "web"
    url_wpcli = "https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar"
    url_bedrock = "https://github.com/roots/bedrock.git"

    config.vm.network "private_network", ip: "#{ip}"
    config.vm.hostname = hostname
    config.vm.box = "scotch/box"
    config.hostsupdater.aliases = ["www.#{name_realhost}","#{name_realhost}"]
    config.vm.synced_folder ".", "#{path_apache_www}", :mount_options => ["dmode=777", "fmode=666"]
    
    # Optional NFS. Make sure to remove other synced_folder line too
    #config.vm.synced_folder ".", "#{path_apache_www}", :nfs => { :mount_options => ["dmode=777","fmode=666"] }

    # to get WordPress up and running
    config.vm.provision "shell", inline: <<-SHELL
        composer self-update
        cd /tmp && sudo wget -q #{url_wpcli} && sudo chmod +x /tmp/wp-cli.phar && sudo mv /tmp/wp-cli.phar /usr/local/bin/wp
        cd #{path_apache_www} && git clone #{url_bedrock} #{name_realhost}
        rm -Rf #{path_apache_www}/#{name_realhost}/.git
        cd #{path_apache_www}/#{name_realhost} && composer update --prefer-dist
        sudo cp #{path_apache_conf}/000-default.conf #{path_apache_conf}/#{name_conf}
        sudo sed -i "s!public!#{name_realhost}/web!g" #{path_apache_conf}/#{name_conf}
        sudo sed -i "s!#ServerName www.example.com!ServerName #{name_realhost}!g" #{path_apache_conf}/#{name_conf}
        sudo a2ensite #{name_realhost}
        sudo service apache2 reload
        printf "DB_NAME=scotchbox\nDB_USER=root\nDB_PASSWORD=root\nDB_HOST=127.0.0.1\nWP_ENV=development\nWP_HOME=http://#{name_realhost}\nWP_SITEURL=http://#{name_realhost}/wp" > #{path_apache_www}/#{name_realhost}/.env
        cd #{path_apache_www}/#{name_realhost}/web && wp core install --allow-root --url=http://#{name_realhost} --title=#{hostname} --admin_user=admin --admin_password=admin --admin_email=admin@#{name_realhost} --quiet
        cd #{path_apache_www}/ && git add #{name_realhost} && git remote remove origin
    SHELL

    config.vm.provider :virtualbox do |vb|
		    vb.gui = false
		    vb.memory = 1024
        vb.name = hostname
        vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
    end

end

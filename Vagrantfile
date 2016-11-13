# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    ### change these two parameters unique to your project!
    #================================================================

    hostname = "scotchbox"
    ip = "192.168.33.10"

    #================================================================

    name_realhost = "#{hostname}.dev"
    name_conf = "#{name_realhost}.conf"
    name_docroot = "web"

    # Apache vhosts root
    path_apache_www = "/var/www"
    # Directory with Apache configs
    path_apache_conf = "/etc/apache2/sites-available"
    # Path to docroot - relative to apache vhosts root
    path_rel_docroot = "#{name_realhost}/#{name_docroot}"
    # Path to docroot - absolute; do not edit this

    url_wpcli = "https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar"
    url_bedrock = "https://github.com/roots/bedrock.git"
    url_wordpress = "http://#{name_realhost}"

    db_name = "scotchbox"
    db_user = "root"
    db_password = "root"
    db_host = "127.0.0.1"

    wp_admin_user = "admin"
    wp_admin_password = "admin"
    wp_admin_email = "#{wp_admin_user}@#{name_realhost}"

    config.vm.network "private_network", ip: "#{ip}"
    config.vm.hostname = hostname
    config.vm.box = "scotch/box"
    config.hostsupdater.aliases = ["www.#{name_realhost}","#{name_realhost}"]
    config.vm.synced_folder ".", "#{path_apache_www}", :mount_options => ["dmode=777", "fmode=666"]
    
    # Optional NFS. Make sure to remove other synced_folder line too
    #config.vm.synced_folder ".", "#{path_apache_www}", :nfs => { :mount_options => ["dmode=777","fmode=666"] }

    # to get WordPress up and running
    config.vm.provision "shell", inline: <<-SHELL
        echo "Updating Composer"
        composer self-update

        echo "Downloading and installing WP CLI"
        cd /tmp && sudo wget -q #{url_wpcli} && sudo chmod +x /tmp/wp-cli.phar && sudo mv /tmp/wp-cli.phar /usr/local/bin/wp

        echo "Downloading and installing Bedrock"
        cd #{path_apache_www} && git clone #{url_bedrock} #{name_realhost}
        rm -Rf #{path_apache_www}/#{name_realhost}/.git

        echo "Downloading dependencies"
        cd #{path_apache_www}/#{name_realhost} && composer update --prefer-dist

        echo "Creating new virtual host with config based on default:"
        echo "#{path_apache_conf}/#{name_conf}"
        sudo cp #{path_apache_conf}/000-default.conf #{path_apache_conf}/#{name_conf}
        sudo sed -i "s!public!#{path_rel_docroot}!g" #{path_apache_conf}/#{name_conf}
        sudo sed -i "s!#ServerName www.example.com!ServerName #{name_realhost}!g" #{path_apache_conf}/#{name_conf}

        echo "Activating new virtual host"
        sudo a2ensite #{name_realhost}
        sudo service apache2 reload
        
        echo "Writing WordPress config"
        printf "DB_NAME=#{db_name}\nDB_USER=#{db_user}\nDB_PASSWORD=#{db_password}\nDB_HOST=#{db_host}\nWP_ENV=development\nWP_HOME=http://#{name_realhost}\nWP_SITEURL=http://#{name_realhost}/wp" > #{path_apache_www}/#{name_realhost}/.env
        echo "Installing WordPress"
        cd #{path_apache_www}/#{path_rel_docroot} && wp core install --allow-root --url=#{url_wordpress} --title=#{hostname} --admin_user=#{wp_admin_user} --admin_password=#{wp_admin_password} --admin_email=#{wp_admin_email} --quiet

        echo "Cleaning up repo"
        cd #{path_apache_www}/ && git add #{name_realhost} && git remote remove origin

        echo "Complete!"
        echo "Go to #{url_wordpress} in your browser"
    SHELL

    config.vm.provider :virtualbox do |vb|
		    vb.gui = false
		    vb.memory = 1024
        vb.name = hostname
        vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
    end

end

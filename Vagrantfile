
Vagrant.configure("2") do |config|
  config.vm.box = "generic/debian10"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 81, host: 8081, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.name   = "lxc.vagrant.vm"
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y lxc lxc-templates
    
        
    export DOWNLOAD_KEYSERVER="hkp://keyserver.ubuntu.com"
    
    
    echo 'USE_LXC_BRIDGE="true"
    LXC_BRIDGE="lxcbr0"
    LXC_ADDR="10.0.3.1"
    LXC_NETMASK="255.255.255.0"
    LXC_NETWORK="10.0.3.0/24"
    LXC_DHCP_RANGE="10.0.3.2,10.0.3.254"
    LXC_DHCP_MAX="253"
    LXC_DHCP_CONFILE=""
    LXC_DOMAIN=""' >> /etc/default/lxc-net

    systemctl enable lxc-net
    systemctl restart lxc-net

    echo 'lxc.net.0.type  = veth
    lxc.net.0.flags = up
    lxc.net.0.link  = lxcbr0' >> /etc/lxc/default.conf

    lxc-create -n C1 -t download -f /etc/lxc/default.conf -- -d centos -r 8-Stream -a amd64 --no-validate
    
    lxc-start C1
    sleep 20
    lxc-attach -n C1 -- yum install -y httpd 
    lxc-attach -n C1 -- systemctl stop firewalld
    lxc-attach -n C1 -- systemctl disable firewalld

    lxc-attach -n C1 -- systemctl enable httpd.service
    lxc-attach -n C1 -- systemctl start httpd.service

    echo "Listen 80" >> /var/lib/lxc/C1/rootfs/etc/httpd/conf/httpd.conf
    echo "<VirtualHost *:80>
      DocumentRoot /var/lib/lxc/C1/rootfs/var/www/php
    </VirtualHost>" >> /var/lib/lxc/C1/rootfs/etc/httpd/conf.d/1.conf
    mkdir /var/lib/lxc/C1/rootfs/var/www/php
    wget https://raw.githubusercontent.com/MaxChernoshey/itacademy-devops-files/master/02-tools/index.html -O /var/lib/lxc/C1/rootfs/var/www/html/index.html
    
    lxc-attach -n C1 -- systemctl reload httpd.service

    lxc-create -n C2 -t download -f /etc/lxc/default.conf -- -d centos -r 8-Stream -a amd64 --no-validate
    
    lxc-start C2
    sleep 20
    lxc-attach -n C2 -- yum install -y php
    lxc-attach -n C2 -- systemctl stop firewalld
    lxc-attach -n C2 -- systemctl disable firewalld

    lxc-attach -n C2 -- systemctl enable httpd.service
    lxc-attach -n C2 -- systemctl start httpd.service


    echo "Listen 81" >> /var/lib/lxc/C2/rootfs/etc/httpd/conf/httpd.conf
    echo "<VirtualHost *:81>
      DocumentRoot /var/lib/lxc/C2/rootfs/var/www/php
    </VirtualHost>" >> /var/lib/lxc/C2/rootfs/etc/httpd/conf.d/1.conf
    mkdir /var/lib/lxc/C2/rootfs/var/www/php
        wget https://raw.githubusercontent.com/MaxChernoshey/itacademy-devops-files/master/02-tools/index.php -O /var/lib/lxc/C2/rootfs/var/www/php/index.php
    
        lxc-attach -n C2 -- systemctl reload httpd.service

  SHELL
  
  config.vm.provision "shell", run: "always", inline: <<-SHELL
  ipc1=$(lxc-ls -f | grep C1 | awk {'print $5'})
  iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 80 -j DNAT --to-destination $ipc1:80
  ipc2=$(lxc-ls -f | grep C2 | awk {'print $5'})
  iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 81 -j DNAT --to-destination $ipc2:81
  SHELL

end




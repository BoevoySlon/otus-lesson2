# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101',
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
            :dfile => './sata2.vdi',
            :size => 250, # Megabytes
		    :port => 2
		},
        :sata3 => {
            :dfile => './sata3.vdi',
            :size => 250,
            :port => 3
        },
        :sata4 => {
            :dfile => './sata4.vdi',
            :size => 250, # Megabytes
            :port => 4
        },
		:sata5 => {
			:dfile => './sata5.vdi', # Путь, по которому будет создан файл диска
			:size => 250, # Размер диска в мегабайтах
			:port => 5 # Номер порта на который будет зацеплен диск
		},

	}

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end

		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end
 	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
	      yum install -y mdadm smartmontools hdparm gdisk
		  mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
		  mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
		  mkdir /etc/mdadm
		  chmod 777 /etc/mdadm
		  echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
		  mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
		  chmod -R 755 /etc/mdadm
		  parted -s /dev/md0 mklabel gpt
		  parted /dev/md0 mkpart primary ext4 0% 20%
		  parted /dev/md0 mkpart primary ext4 20% 40%
		  parted /dev/md0 mkpart primary ext4 40% 60%
		  parted /dev/md0 mkpart primary ext4 60% 80%
		  parted /dev/md0 mkpart primary ext4 80% 100%
		  for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
		  mkdir -p /raid/part{1,2,3,4,5}
		  for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
		  df -h
  	  SHELL

      end
  end
end
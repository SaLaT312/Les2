# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
      :box_name => "centos/7",              # Название базового бокса Vagrant
      :ip_addr => '192.168.11.101',         # IP адрес серой сети
	  :disks => {                           #Подключение дополнительных дисков
		
        :sata1 => {                         # Создание дополнительного Sata носителя №1
			:dfile => './sata1.vdi',        # Имя носителя
			:size => 250,                   # Размер в мегабайтах
			:port => 1                      # Номер порта для подключения
		    },
		:sata2 => {                         # Создание дополнительного Sata носителя №2
            :dfile => './sata2.vdi',
            :size => 250, 
			:port => 2
    		},
        :sata3 => {                         # Создание дополнительного Sata носителя №3
            :dfile => './sata3.vdi',
            :size => 250,
            :port => 3
            },
        :sata4 => {                         # Создание дополнительного Sata носителя №4
            :dfile => './sata4.vdi',
            :size => 250, 
            :port => 4
            },
        :sata5 => {                         # Создание дополнительного Sata носителя №5
            :dfile => './sata5.vdi',
            :size => 250, 
            :port => 5
            }

#        :sata6 => {                         # Создание дополнительного Sata носителя №6 для зеркала системы
#            :dfile => './sata6.vdi',
#            :size => 40960, 
#            :port => 6
#            }

	}

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.box_check_update = false

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', 
                                '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
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
# Выполнение скрипта если система запущена впервые
        if [ -f "/var/log/non_first" ]
         then 
echo "Система готова к использованию"
        else
        {
#sudo -i
# Запись публичных ключей и установка дополнительных пакетов
	      mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
	      yum install -y mdadm smartmontools hdparm gdisk nano
          echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDLZNA8U+LU480dKadym1PxAzhUD/7G6Q2i67jz3xNNJpGFIlsU1bkh6xIvMvaVyrN1ZjLpvVlRYqpu9r6cKzquf7yiq+u8ImTDRtIVFEKXDF+QML07QwM1ktJg7/ZBnlMC/W1gIxPUUxJ5aScLkr+zigBQyL+DkEOIVZWs9dJEDOdUNHe2mm3dXKCtnzD2uWwlZX6Dz6HOqJkgIZm5QCJSk4Hg/LKMn7NafmbPHweMyE9WoJY4TmSINsAC3NyERdbkdY8cWTd7KoDd9bE+mIPqKk26K/jWyEZb+k5qi14b8jfxPPby6Y99zH2Fr4HP5gh1ho5cG51/d4854nEzo+l7 salat@salat-MS" >> /root/.ssh/authorized_keys   



# Создание GPT на sdb и разбивка на партиции
parted -s  /dev/sdb mklabel gpt
parted -s  /dev/sdb mkpart primary ext4 0% 20% 
parted -s  /dev/sdb mkpart primary ext4 20% 40%
parted -s  /dev/sdb mkpart primary ext4 40% 60%
parted -s  /dev/sdb mkpart primary ext4 60% 80%
parted -s  /dev/sdb mkpart primary ext4 80% 100%


# Создание GPT на sdc и разбивка на партиции
parted -s  /dev/sdc mklabel gpt
parted -s  /dev/sdc mkpart primary ext4 0% 20% 
parted -s  /dev/sdc mkpart primary ext4 20% 40%
parted -s  /dev/sdc mkpart primary ext4 40% 60%
parted -s  /dev/sdc mkpart primary ext4 60% 80%
parted -s  /dev/sdc mkpart primary ext4 80% 100%


echo "yes" |mdadm --zero-superblock --force /dev/sd{b,c,d,e,f} #зануление суперблоков дисков
# Создание блочного устройства md0 Raid5 из устройств sdd, sde и sdf
echo "yes" | mdadm --create --verbose /dev/md0 -l 5 -n 3 /dev/sd{d,e,f}
# Создание блочных из устройств mdX Raid1 из устройств sdb и sdc
echo "yes" | mdadm --create --verbose /dev/md1 -l 1 -n 2 /dev/sd{b,c}1
echo "yes" | mdadm --create --verbose /dev/md2 -l 1 -n 2 /dev/sd{b,c}2 
echo "yes" | mdadm --create --verbose /dev/md3 -l 1 -n 2 /dev/sd{b,c}3 
echo "yes" | mdadm --create --verbose /dev/md4 -l 1 -n 2 /dev/sd{b,c}4 
echo "yes" | mdadm --create --verbose /dev/md5 -l 1 -n 2 /dev/sd{b,c}5 

# Форматирование разделов в ext4
mkfs.ext4 /dev/md1  
mkfs.ext4 /dev/md1  
mkfs.ext4 /dev/md2
mkfs.ext4 /dev/md3
mkfs.ext4 /dev/md4
mkfs.ext4 /dev/md5         
        
#Создание точек монтирования
mkdir /mnt/{1,2,3,4,5}

# Прописываем все в fstab
echo " `blkid |grep md1 | awk '{print $2}'`        /mnt/1    ext4    defaults    0 0" >> /etc/fstab
echo " `blkid |grep md2 | awk '{print $2}'`        /mnt/2    ext4    defaults    0 0" >> /etc/fstab
echo " `blkid |grep md3 | awk '{print $2}'`        /mnt/3    ext4    defaults    0 0" >> /etc/fstab
echo " `blkid |grep md4 | awk '{print $2}'`        /mnt/4    ext4    defaults    0 0" >> /etc/fstab
echo " `blkid |grep md5 | awk '{print $2}'`        /mnt/5    ext4    defaults    0 0" >> /etc/fstab

# Монтируем все устройства
mount -a


# Завершаем первую преднастройку 
touch /var/log/non_first
}
fi        
  	  SHELL

      end
  end
end


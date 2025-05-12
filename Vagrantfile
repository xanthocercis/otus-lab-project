Vagrant.configure("2") do |config|
  # Создаем директорию для дисков если не существует
  FileUtils.mkdir_p("D:/vagrant_disks") unless File.directory?("D:/vagrant_disks")

  # Определение 4 ВМ
  (1..6).each do |i|
    config.vm.define "vm#{i}" do |vm|
      # Используем бокс Oracle Linux 9
      vm.vm.box = "oraclelinux/9"

      # Настройка приватной сети с IP-адресами 192.168.56.11-14
      vm.vm.network "private_network", ip: "192.168.56.#{10 + i}"

      # Настройка провайдера VirtualBox
      vm.vm.provider "virtualbox" do |vb|
        vb.memory = 2048  # 2 ГБ оперативной памяти
        vb.cpus = 1       # 1 CPU
        vb.name = "vm#{i}"

        # Создание дополнительного диска на 15 ГБ
        disk_file = "D:/vagrant_disks/vm#{i}_disk.vdi"
        unless File.exist?(disk_file)
          vb.customize ['createhd', '--filename', disk_file, '--size', 15 * 1024]
        end
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', 
                     '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk_file]
      end

      vm.vm.provision "shell", inline: <<-SHELL
        mkdir -p /home/vagrant/templates
        chown vagrant:vagrant /home/vagrant/templates
        chmod 755 /home/vagrant/templates
        echo "nameserver 8.8.8.8" > /etc/resolv.conf
        echo "nameserver 8.8.4.4" >> /etc/resolv.conf
        sudo firewall-cmd --permanent --add-port=10050/tcp
        sudo firewall-cmd --reload
      SHELL

      # Копирование общих файлов
      vm.vm.provision "file", source: "./base_playbook.yml", destination: "/home/vagrant/"
      vm.vm.provision "file", source: "./playbook_vm#{i}.yml", destination: "/home/vagrant/"
      vm.vm.provision "file", source: "./promtail-config.yaml.j2", destination: "/home/vagrant/"
      
      # Копирование специфических файлов для каждой ВМ
      case i
      when 1
        vm.vm.provision "file", source: "./templates/vm1/wordpress.conf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm1/wp-config.php.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm5/zabbix_agent2.conf.j2", destination: "/home/vagrant/templates/"
      when 2
        vm.vm.provision "file", source: "./templates/vm2/my.cnf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm5/zabbix_agent2.conf.j2", destination: "/home/vagrant/templates/"
      when 3
        vm.vm.provision "file", source: "./templates/vm3/my-slave.cnf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm5/zabbix_agent2.conf.j2", destination: "/home/vagrant/templates/"
      when 4
        vm.vm.provision "file", source: "./templates/vm4/zabbix_server.conf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm4/filebeat.yml", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm2/my.cnf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm4/nginx.conf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm4/php-fpm.conf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm5/zabbix_agent2.conf.j2", destination: "/home/vagrant/templates/"
      when 5
        vm.vm.provision "file", source: "./templates/vm5/wordpress.conf.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm5/wp-config.php.j2", destination: "/home/vagrant/templates/"
        vm.vm.provision "file", source: "./templates/vm5/zabbix_agent2.conf.j2", destination: "/home/vagrant/templates/"
      when 6
        vm.vm.provision "file", source: "./templates/vm5/zabbix_agent2.conf.j2", destination: "/home/vagrant/templates/"
      end

      # Установка зависимостей
      vm.vm.provision "shell", inline: <<-SHELL
        sudo dnf install -y epel-release
        sudo dnf install -y ansible git
        sudo dnf install -y python3-pip python3-PyMySQL
        sudo dnf install -y mariadb-server python3-PyMySQL python3-pip
        sudo python3 -m pip install pymysql --upgrade

      SHELL

      # Выполнение плейбуков
      vm.vm.provision "shell", inline: <<-SHELL
        ansible-playbook /home/vagrant/base_playbook.yml
        ansible-playbook /home/vagrant/playbook_vm#{i}.yml
      SHELL

  # Копирование шаблона конфигурации
  # Копирование и запуск плейбука
  vm.vm.provision "file", source: "./playbook_vm2.yml", destination: "/home/vagrant/"
  vm.vm.provision "shell", inline: "ansible-playbook /home/vagrant/playbook_vm2.yml"
    end
  end
end
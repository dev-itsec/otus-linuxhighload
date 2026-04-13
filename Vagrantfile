BOX = "ubuntu/jammy64"

NODES = {
  "pg-node1" => {
    ram: 512,
    cpu: 2,
    networks: [
      { ip: "10.20.30.41", virtualbox__intnet: "lan" },
      { ip: "192.168.56.11", adapter: 8 }
    ]
  }, 	
  "pg-node2" => {
    ram: 512,
    cpu: 2,
    networks: [
      { ip: "10.20.30.42", virtualbox__intnet: "lan" },
      { ip: "192.168.56.12", adapter: 8 }
    ]
  },
  "pg-node3" => {
    ram: 1024,
    cpu: 2,
    networks: [
      { ip: "10.20.30.43", virtualbox__intnet: "lan" },
      { ip: "192.168.56.13", adapter: 8 }
    ]
  },
  "proxy-node1" => {
    ram: 1024,
    cpu: 2,
    networks: [
      { ip: "10.20.30.44", virtualbox__intnet: "lan" },
      { ip: "192.168.56.14", adapter: 8 }
    ]
  },
  "proxy-node2" => {
    ram: 1024,
    cpu: 2,
    networks: [
      { ip: "10.20.30.45", virtualbox__intnet: "lan" },
      { ip: "192.168.56.15", adapter: 8 }
    ]
  },
  "web-node1" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.46", virtualbox__intnet: "lan" },
      { ip: "192.168.56.16", adapter: 8 }
    ]
  },
  "web-node2" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.47", virtualbox__intnet: "lan" },
      { ip: "192.168.56.17", adapter: 8 }
    ]
  },
  "ceph-node1" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.48", virtualbox__intnet: "lan" },
      { ip: "192.168.56.18", adapter: 8 }
    ],
    disk_size: 10240  
  },
  "ceph-node2" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.49", virtualbox__intnet: "lan" },
      { ip: "192.168.56.19", adapter: 8 }
    ],
    disk_size: 10240  
  },
  "ceph-node3" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.50", virtualbox__intnet: "lan" },
      { ip: "192.168.56.20", adapter: 8 }
    ],
    disk_size: 10240  
  },
  "es-node1" => {
    ram: 3072,
    cpu: 2,
    networks: [
      { ip: "10.20.30.51", virtualbox__intnet: "lan" },
      { ip: "192.168.56.21", adapter: 8 }
    ]
  },
  "es-node2" => {
    ram: 3072,
    cpu: 2,
    networks: [
      { ip: "10.20.30.52", virtualbox__intnet: "lan" },
      { ip: "192.168.56.22", adapter: 8 }
    ]
  },
  "es-node3" => {
    ram: 3072,
    cpu: 2,
    networks: [
      { ip: "10.20.30.53", virtualbox__intnet: "lan" },
      { ip: "192.168.56.23", adapter: 8 }
    ] 
  },
  "logstash-node1" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.54", virtualbox__intnet: "lan" },
      { ip: "192.168.56.24", adapter: 8 }
    ] 
  },
  "logstash-node2" => {
    ram: 2048,
    cpu: 2,
    networks: [
      { ip: "10.20.30.55", virtualbox__intnet: "lan" },
      { ip: "192.168.56.25", adapter: 8 }
    ] 
  },
  "prom-node1" => {
    ram: 1024,
    cpu: 2,
    networks: [
      { ip: "10.20.30.56", virtualbox__intnet: "lan" },
      { ip: "192.168.56.26", adapter: 8 }
    ] 
  },
  "prom-node2" => {
    ram: 1024,
    cpu: 2,
    networks: [
      { ip: "10.20.30.57", virtualbox__intnet: "lan" },
      { ip: "192.168.56.27", adapter: 8 }
    ] 
  }  
}

Vagrant.configure("2") do |config|
  config.vm.box = BOX
  config.vm.boot_timeout = 900
  config.vm.synced_folder ".", "/vagrant", disabled: true
 
  NODES.each do |name, opts|
    config.vm.define name do |node|
      node.vm.hostname = name
      
      node.vm.provider "virtualbox" do |vb|
        vb.name = name
        vb.memory = opts[:ram]
        vb.cpus = opts[:cpu]
        
        # Добавление второго диска для ceph-node*
        if opts[:disk_size]
          disk_file = File.join(Dir.home, "VirtualBox VMs", name, "#{name}_disk2.vmdk")       
          unless File.exist?(disk_file)
            vb.customize ['createmedium', 'disk', '--filename', disk_file, '--size', opts[:disk_size], '--format', 'vmdk']
          end  
          vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', disk_file]
        end
      end
      
      # Networks
      opts[:networks].each do |net|
        node.vm.network "private_network", **net
      end
      
      # Provisioning для всех нод
      node.vm.provision "shell", inline: <<-SHELL
        timedatectl set-timezone Europe/Moscow
        echo "ubuntu:1234" | chpasswd
        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
        sudo systemctl restart sshd        
        apt update
        apt-get install -y mc net-tools tcpdump
        echo "Sleeping 5 seconds to slow down provisioning sequence..."
        sleep 5
      SHELL
      
      # Дополнительный provisioning для ceph-node*
      if name.start_with?('ceph-node')
        node.vm.provision "shell", inline: <<-SHELL
          apt-get install -y docker.io
          echo "Sleeping 5 seconds for ceph-node..."
          sleep 5
        SHELL
      end
    end
  end
end
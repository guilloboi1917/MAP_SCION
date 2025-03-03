Vagrant.configure("2") do |config|
    vm_count = 5
    base_ip = "192.168.56."

    config.vm.box = "ubuntu/jammy64" # Ubuntu 22.04
  
    (1..vm_count).each do |i|
      config.vm.define "scion0#{i}" do |node|
        # Define main settings
        node.vm.hostname = "scion0#{i}"
        node.vm.network "private_network", ip: "#{base_ip}#{10 + i}"
  
        node.vm.provider "virtualbox" do |vb|
          vb.memory = 1024
          vb.cpus = 1
        end
        
        # Config ssh
        node.vm.provision "shell", inline: <<-'SHELL'
          sed -i 's/^#* *\(PermitRootLogin\)\(.*\)$/\1 yes/' /etc/ssh/sshd_config
          sed -i 's/^#* *\(PubkeyAuthentication\)\(.*\)$/\1 yes/' /etc/ssh/sshd_config
          sudo systemctl restart sshd
        SHELL

        # VM 1 generates an SSH key and stores it in the shared folder
        if i == 1
          node.vm.provision "shell", privileged:false, inline: <<-SHELL
            if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
              ssh-keygen -t rsa -N "" -f /home/vagrant/.ssh/id_rsa
            fi
            cp /home/vagrant/.ssh/id_rsa.pub /vagrant/id_rsa_1.pub
            sudo systemctl restart sshd
          SHELL
        else
          # Other VMs wait for VM 1's public key and add it to their authorized_keys
          node.vm.provision "shell", inline: <<-SHELL
            while [ ! -f /vagrant/id_rsa_1.pub ]; do
              echo "Waiting for VM 1's public key..."
              sleep 2
            done
            cat /vagrant/id_rsa_1.pub >> /home/vagrant/.ssh/authorized_keys
            chmod 600 /home/vagrant/.ssh/authorized_keys
            chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
            sudo systemctl restart sshd
          SHELL
        end

        node.vm.provision "shell", privileged: true, path: "setup_scion.sh", env: { "VM_INDEX" => i.to_s }
        node.vm.provision "file", source: "topology_files/topology#{i}.json", destination: "/tmp/topology.json" 

        # Reboot after this step
        node.vm.provision "shell", reboot:true , inline: "sudo mv /tmp/topology.json /etc/scion/topology.json"

        # For first machine setup certificates to be distributed later
        if i == 1
          node.vm.provision "shell", privileged: true, path: "certificate_setup.sh"
        end

        # Copy generated certs and start services
        node.vm.provision "shell", 
          env: { "VM_INDEX" => i.to_s },
          inline: <<-SHELL
            # Ensure directories exist
            sudo mkdir -p /etc/scion/crypto/as
            sudo mkdir -p /etc/scion/certs

            # Copy certificates
            sudo cp /vagrant/tutorial-scion-certs/AS${VM_INDEX}/cp-as.key /etc/scion/crypto/as/
            sudo cp /vagrant/tutorial-scion-certs/AS${VM_INDEX}/cp-as.pem /etc/scion/crypto/as/
            sudo cp /vagrant/tutorial-scion-certs/ISD42-B1-S1.trc /etc/scion/certs/

            # Start SCION services
            sudo systemctl start scion-router@br.service
            sudo systemctl start scion-control@cs.service
            sudo systemctl start scion-daemon.service
            
            # Check that all services are active
            systemctl status scion-*.service
          SHELL
      end
    end
end
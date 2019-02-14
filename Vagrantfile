# -*- mode: ruby -*-
# # vi: set ft=ruby :

Vagrant.configure(2) do |config|

  (1..3).each do |i|
    config.vm.define "k8s#{i}" do |s|
      s.ssh.forward_agent = true
      s.vm.box = "ubuntu/xenial64"
      s.vm.hostname = "k8s#{i}"
      s.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
      if Vagrant.has_plugin?("vagrant-vbguest")
        config.vbguest.auto_update = false
      end
      s.vm.network "private_network", ip: "172.42.42.#{i+10}", netmask: "255.255.255.0",
                   auto_config: true,
                   virtualbox__intnet: "k8s-net"

      s.vm.provider "virtualbox" do |v|
        v.name = "k8s#{i}"
        v.memory = 1500
        v.gui = false
      end

      if i == 1
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-master.yml -c local"
      else
        #joinCommand = system("vagrant ssh k8s1 -c 'sudo kubeadm token create --print-join-command'")
        # s.vm.provision :shell, inline: "ansible all -i 172.42.42.11, -m copy -a 'src=~/joinCommand.txt dest=/tmp owner=ubuntu group=ubuntu'"
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-worker.yml -c local"
        s.vm.provision :shell,
        run: "always",
        inline: "echo Setting Cluster Route; clustip=$(kubectl --kubeconfig=/vagrant/admin.conf get svc -o json | JSONPath.sh '$.items[?(@.metadata.name=kubernetes)]..clusterIP' -b); node=$(kubectl --kubeconfig=/vagrant/admin.conf get endpoints -o json | JSONPath.sh '$.items[?(@.metadata.name=kubernetes)]..ip' -b); ip route add $clustip/32 via $node || true"
      end
      # if i != 1
      #   # Add a route back to the kubernetes API service
      #   s.vm.provision :shell,
      #   run: "always",
      #   inline: "echo Setting Cluster Route; clustip=$(kubectl --kubeconfig=/vagrant/admin.conf get svc -o json | JSONPath.sh '$.items[?(@.metadata.name=kubernetes)]..clusterIP' -b); node=$(kubectl --kubeconfig=/vagrant/admin.conf get endpoints -o json | JSONPath.sh '$.items[?(@.metadata.name=kubernetes)]..ip' -b); ip route add $clustip/32 via $node || true"
      # end
      if s.vm.hostname == "k8s1"
        s.vm.network "forwarded_port", guest: 443, host: "4431"
        s.vm.network "forwarded_port", guest: 80, host: "1800"
        s.vm.network "forwarded_port", guest: 8001, host: "8001"
        s.vm.network "forwarded_port", guest: 30399, host: "30399"
        s.vm.network "forwarded_port", guest: 1234, host: "11234"
        s.vm.network "forwarded_port", guest: 5000, host: "5000"
        s.vm.network "forwarded_port", guest: 6443, host: "6443"
      elsif s.vm.hostname == "k8s2"
        s.vm.network "forwarded_port", guest: 443, host: "4432"
        s.vm.network "forwarded_port", guest: 80, host: "2800"
        s.vm.network "forwarded_port", guest: 8001, host: "8002"
        s.vm.network "forwarded_port", guest: 1234, host: "21234"
      elsif s.vm.hostname == "k8s3"
        s.vm.network "forwarded_port", guest: 443, host: "4433"
        s.vm.network "forwarded_port", guest: 80, host: "3800"
        s.vm.network "forwarded_port", guest: 8001, host: "8003"
        s.vm.network "forwarded_port", guest: 1234, host: "31234"
      end

    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

end

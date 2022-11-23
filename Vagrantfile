master = 1
node = 1
nodes = []

Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  (1..(master + node)).each do |machine_id|
    name = (machine_id <= master) ? "master" : "node"
    id = (machine_id <= master) ? machine_id : (machine_id - master)
    
    if name == "node"
      nodes.append("k8s_#{name}_#{id}")
    end

    config.vm.define "k8s_#{name}_#{id}" do |machine|
      machine.vm.hostname = "k8s-#{name}-#{id}"
      machine.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh", disabled: true
      machine.vm.provider :vmware_fusion do |vmware|
          vmware.vmx["numvcpus"] = 2
          vmware.vmx["memsize"] = 4096
      end
      machine.vm.provider :virtualbox do |vbox|
        vbox.customize ["modifyvm", :id, "--nic1", "nat"]
        vbox.cpus = 2
        vbox.memory = 4096
      end

      if machine_id == (master + node)
        machine.vm.provision :ansible do |ansible|
          ansible.groups = {
            "master" => ["k8s_master_1"],
            "nodes" => nodes,
            "kube_cluster:children" => ["master", "nodes"]
          }
          ansible.limit = "all"
          ansible.playbook = "build/playbook.yml"
        end
      end
    end
  end
end

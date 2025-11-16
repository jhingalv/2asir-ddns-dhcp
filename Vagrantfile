Vagrant.configure("2") do |config|

  config.vm.box = "debian/bookworm64"

  # DNS SERVER
  config.vm.define "dns" do |dns|
    dns.vm.hostname = "dns"
    dns.vm.network "private_network",
      ip: "192.168.58.10",
      virtualbox__intnet: "ddns"
    dns.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/site.yml"
      ansible.inventory_path = "ansible/inventory"
      ansible.limit = "dns"
    end
  end

  # DHCP SERVER
  config.vm.define "dhcp" do |dhcp|
    dhcp.vm.hostname = "dhcp"
    dhcp.vm.network "private_network",
      ip: "192.168.58.20",
      virtualbox__intnet: "ddns"
    dhcp.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/site.yml"
      ansible.inventory_path = "ansible/inventory"
      ansible.limit = "dhcp"
    end
  end

  # CLIENT
  config.vm.define "client" do |client|
    client.vm.hostname = "client"
    client.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "ddns"
    client.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/site.yml"
      ansible.inventory_path = "ansible/inventory"
      ansible.limit = "client"
    end
  end

end

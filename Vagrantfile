Vagrant.configure("2") do |config|
config.vm.box = "centos/7"
config.vm.box_version = "2004.01"
config.vm.provider :virtualbox do |v|
v.memory = 2048
v.cpus = 2
end
# Define two VMs with static private IP addresses.
boxes = [
{ :name => "web",
:ip => "192.168.56.10",
},
{ :name => "log",
:ip => "192.168.56.15",
},
{ :name => "elk",
:ip => "192.168.56.25",
}
]
# Provision each of the VMs.
boxes.each do |opts|
config.vm.define opts[:name] do |config|
config.vm.hostname = opts[:name]
config.vm.network "private_network", ip: opts[:ip]
if opts[:name] == "elk"
config.vm.network "forwarded_port", guest: 5601, host: 5601
end
if opts[:name] == boxes.last[:name]
config.vm.provision "ansible" do |ansible|
ansible.playbook = "ansible/main.yaml"
ansible.become = "true"
ansible.host_key_checking = "false"
ansible.limit = "all"
end
end
end
end
end

title: Provissioning Jenkins with Vagrant
date: 2020-10-02
tags: [jenkins, vagrant]

---
99 percent of the time our work revolves around surfing the internet in search of solutions for all the day to day hassles. All the contents that I surf mostly deals with basic DevOps and Cloud Orchestration Solutions,  but for more complex problems there seems to be no single website that could explain a SRE thirst for best practices that we could approach in a more professional way in our daily work life.

This site intends to do just that. I prefer to dive in straight to the point where it matters. Here I wish upon building right from setting up a development environment that will be used to build more complex solutions in the future.   At the time of starting this blog, my development environment looks like the below:



**Machine**: A modest HP Pavilion, i7 10th Gen Processor  Win10 with 16 Gigs of RAM and 512 Gigs of SSD.

**Virtualizations**: VirtualBox + Vagrant with CentOS 7.8 Boxes. ( Dont bother about the versions. All latest versions is just more than sufficient to work). For more information on the installation and setting up  VirtualBox and Vagrant, check the links below:

*`VirtualBox`*: https://www.virtualbox.org/manual/ch02.html

`Vagrant`: https://www.vagrantup.com/docs/installation

`Vagrant Images`: https://app.vagrantup.com/bento/boxes/centos-7.8

`GitHuB`: https://github.com/manjunathrreddy/

Vagrantfile to Provision VMs locally as below:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

  Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.ssh.insert_key = false
    config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
  end

#Master VM
  config.vm.define "cicd-admin-server" do |cicd-admin-server|
    cicd-admin-server.vm.hostname = "master.server.dev"
    cicd-admin-server.vm.box = "bento/centos-7.8"
    cicd-admin-server.vm.network :private_network, ip: "192.168.60.10"
  end

# Node VM - 1
  config.vm.define "node1" do |node1|
    node1.vm.hostname = "node1.server.dev"
    node1.vm.box = "bento/centos-7.8"
    node1.vm.network :private_network, ip: "192.168.60.12"
  end

#Node VM - 2

  config.vm.define "node2" do |node2|
    node2.vm.hostname = "node2.server.dev"
    node2.vm.box = "bento/centos-7.8"
    node2.vm.network :private_network, ip: "192.168.60.14"
  end
end
```

The above vagrant script creates a master cicd server for all admin and cloud orchestration, and the 2 nodes to be as slaves machines for load sharing.

The above Vagrantfile will be further refined accordingly for specific use cases in future. From the directory where the Vagrant file is present. Trigger the Below command

`$vagrant up cicd-admin-server node1 node2`

Brings all the vms to life!!!

Now if you are wondering what about Docker and Kubernetes, thats a different eco space for microservices. I prefer using any one of the managed services (preferably GKE).
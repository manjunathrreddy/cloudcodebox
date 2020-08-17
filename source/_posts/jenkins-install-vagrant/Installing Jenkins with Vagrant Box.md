We will further install a Jenkins Instance while initializing a vagrant box. The refined Vagrant and the related jenkins installation is as below:

Jenkins.sh should be at the same level as vagrantfile

```shell
sudo yum install -y epel-release
sudo yum -y update
sudo yum install -y net-tools
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install -y git 
sudo yum install -y java-1.8.0-openjdk.x86_64
sudo yum -y install jenkins 
sudo systemctl start jenkins
sudo systemctl enable jenkins
echo "Installing Jenkins Plugins"
JENKINSPWD=`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
echo $JENKINSPWD
```



Our refined Vagrantfile code is as below

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
    	cicdserver.vm.provision "shell" do |shell|
    	shell.path = "jenkins.sh"
  		end
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



Now while we try to hit the URL: http://192.168.60.10:8080/



![image-20200817212213660](D:\Data\Personal_blog_environment\cloud-garage-blog\source\_posts\image_jenkins\image-20200817212213660.png)



Apply the password that is printed during the vagrant spinning up the vm.
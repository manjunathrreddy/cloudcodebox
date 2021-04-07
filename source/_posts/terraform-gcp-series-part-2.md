---
title: Terraform GCP Series - Part 2: Making Terraform Dynamic with Interpolation
date: 2020-11-07 14:06:49
tags: [Terraform, GCP]
---

Continuing from the previous post we will try to introduce interpolation, flow control and looping. We will split the main.tf to different chunks of files that hold specific definitions to create the resources in GCP. We will create the **provider.tf** file which holds the provider configurations. 

*provider.tf*

```
variable "path" {  default = "/home/vagrant/gcp_credentials/keys" }

provider "google" {
    project = "triple-virtue-271517"
    version = "~> 3.38.0"
    region = "us-central1"
    zone = "us-central1-a"
    credentials = "${file("${var.path}/triple-virtue.json")}"
  
}
```

Firewall rules can be defined in a separate file as **firewall.tf** as below:

*firewall.tf*

```
resource "google_compute_firewall" "allow-http-port" {
    name = "allow-http-port"
    network = "default"

    allow {
        protocol = "tcp"
        ports = ["80"]
    }

    target_tags = ["allow-http"]
      
}

resource "google_compute_firewall" "allow-https-port" {
    name = "allow-https-port"
    network = "default"

    allow {
        protocol = "tcp"
        ports = ["443"]
    }

    target_tags = ["allow-https"]
      
}
```



Interpolation in Terraform helps to assign values to variables, this way we can dynamically manage the provisioning of resources in the cloud environments. Here we create **variables.tf** file with defines the variables that can be used in the script.

*variable.tf*

```
variable "image" {  default = "centos-8" }
variable "machine_type" { default = "n1-standard-2" }
variable "name_count" { default = ["server-1","server-2","server-3"]}
variable "environment" { default = "production" }
variable "machine_type_dev" { default = "n1-standard-1" }
variable "machine_count" { default = "1" }
variable "machine_size" { default = "20" }
```



We will then create a seperate file **httpd_install.sh** where we install the web servers into the compute instances.

*httpd_install.sh*

```
sudo yum update 
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```



Now lets define the **main.tf**  that reffers to the interpolation, firewall rules and the script to install the apache webservers.

*main.tf*

```
resource "google_compute_instance" "default" {
    
    count = length(var.name_count)
    name = "list-${count.index+1}"
    machine_type = var.environment != "production" ? var.machine_type : var.machine_type_dev
    metadata_startup_script = "${file("httpd_install.sh")}" 
    

    can_ip_forward = "false"
    description = "This is our virtual machines"

        tags = ["allow-http","allow-https"]

  


    boot_disk {
        initialize_params {
            image = var.image
            size = var.machine_size
        }
    }


    network_interface {
        network = "default"
        access_config {
      // Ephemeral IP
        }
    }

    metadata = {
        size = "20"
        foo = "bar"
    }

}
```

Now by carefully observing main.tf, we see that the lines refer to the variables defined in the *variables.tf* 

```
    count = length(var.name_count)
    name = "list-${count.index+1}"
    machine_type = var.environment != "production" ? var.machine_type : var.machine_type_dev
```

Further the above lines also shows the looping and flow control. Here  we are looping to create 3 compute instances of type production grade. Below we see clear interpolation the terraform which refers the *image* and *machine_size* defined in the variables.tf

    boot_disk {
        initialize_params {
            image = var.image
            size = var.machine_size
        }
    }


The below line initializes the installation of apache webservers with httpd_install.sh script.

` metadata_startup_script = "${file("httpd_install.sh")}"`



Hence the **output.tf** will look like below:

*output.tf*

```
output "machine_type" {
  value = "${google_compute_instance.test_instance[*].machine_type}"
}

output "name" {
  value = "${google_compute_instance.test_instance[*].name}"
}
```

 

the overall files created in this regard is as below:



```
---/gce/
	-- firewall.tf
	-- httpd_install.sh
	-- main.tf
	-- output.tf
	-- provider.tf
	-- variables.tf	
```



The results of the above experiments are as below:



{% asset_img server_instances.JPG %}




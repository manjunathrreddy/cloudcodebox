---
title: Orchestrate Multiple Environments with GCP
date: 2021-05-25 22:13:23
tags: [Terraform, GCP, Terraform GCP Series]
---

The purpose of this series of posts on Terraform with GCP is to accomplish more with less.  Here we try to optimize our templates for bringing up multiple environments across multple projects in GCP.  Below approach will help spin multiple instances with minimal efforts by introducing .tfvars files into our templates.

***Use case***: I have 2  projects gcp-homecompany-qa and gcp-homecompany-dev for this purpose and we will have to create compute instances with terraform on GCP. Lets get on with it.



The folder structure goes as below

```
---/gce/`
	-- firewall.tf
	-- httpd_install.sh
	-- main.tf
	-- output.tf
	-- provider.tf
	-- variables.tf
	-- app-dev.tfvars
    -- app-qa.tfvars
```

The ***varaibles.tf*** file will be used to declare the variables and to assign few default settings

```
variable "test_servers" { 

  type = list(any) 
}

variable "disk_zone" { default = "" }
variable "disk_type" { default = "" }
variable "disk_name" { default = "" }
variable "disk_size" { default = "" }
variable "project_id" { default = "" }
variable "credentials_file" { default = "" }
variable "path" { default = "/home/admin/gcp_credentials/keys" }
```



The values to these variables are assigned in the respective ***.tfvars*** files, so here we create 2 .tfvars files to lets say spin up 2 environments **Dev** and ***QA*** environments. And the two files are defined as below:

*dev.tfvars* 

```
credentials_file = "gcp-homecompany-dev-key.json"

project_id = "gcp-homecompany-dev"



test_servers = [{

  id = 1

  compute_instance_name = "demo1"

  compute_machine_type = "e2-standard-2"

  compute_image = "centos-8"

  compute_network = "home-network"

  compute_subnet = "home-sub-subnetwork"

  compute_zone = "us-central1-a"

  compute_size = "100"

},

{

  id = 2

  compute_instance_name = "demo2"

  compute_machine_type = "e2-standard-2"

  compute_image = "centos-8"

  compute_network = "home-network"

  compute_subnet = "home-sub-subnetwork"

  compute_zone = "us-central1-a"

  compute_size = "100"

},

{

  id = 3

  compute_instance_name = "demo3"

  compute_machine_type = "e2-standard-2"

  compute_image = "centos-8"

  compute_network = "home-network"

  compute_subnet = "home-sub-subnetwork"

  compute_zone = "us-central1-a"

  compute_size = "100"

}]



disk_zone = "us-east1-b"

disk_type = "pd-ssd"

disk_name = "additional volume disk"

disk_size = "150"
```



*app-qa.tfvars*

```
credentials_file = "gcp-homecompany-qa-key.json"

project_id = "gcp-homecompany-qa"



test_servers = [{

  id = 1

  compute_instance_name = "demo1"

  compute_machine_type = "e2-standard-2"

  compute_image = "centos-8"

  compute_network = "home-network"

  compute_subnet = "home-sub-subnetwork"

  compute_zone = "us-central1-a"

  compute_size = "100"

},

{

  id = 2

  compute_instance_name = "demo2"

  compute_machine_type = "e2-standard-2"

  compute_image = "centos-8"

  compute_network = "home-network"

  compute_subnet = "home-sub-subnetwork"

  compute_zone = "us-central1-a"

  compute_size = "100"

},

{

  id = 3

  compute_instance_name = "demo3"

  compute_machine_type = "e2-standard-2"

  compute_image = "centos-8"

  compute_network = "home-network"

  compute_subnet = "home-sub-subnetwork"

  compute_zone = "us-central1-a"

  compute_size = "100"

}]

disk_zone = "us-east1-b"

disk_type = "pd-ssd"

disk_name = "additional volume disk"

disk_size = "150"
```



In the above *.tfvars* files we tried to populate the list **test_servers** with 3 google compute instances. In order to iterate through this list with key-value pairs we try to implement for loop with a *for_each* meta-arguments in the below template. Hence following changes are to be done to our *main.tf* file:



```
resource "google_compute_instance" "test_instance" {

  for_each = { for test_instance in var.test_servers : test_instance.id => test_instance }

  name = each.value.compute_instance_name

  machine_type = each.value.compute_machine_type

  zone = each.value.compute_zone

  metadata_startup_script = "${file("httpd_install.sh")}" 

  can_ip_forward = "false"



//  tags = ["",""]

  description = "This is our virtual machines"
  tags = ["allow-http","allow-https"]
  boot_disk {
    initialize_params {
      image = each.value.compute_image
      size = each.value.compute_size
    }
  }


  network_interface {
    network = each.value.compute_network
    subnetwork = each.value.compute_subnet
    access_config {
   // Ephemeral IP
    }
  }

 service_account {
  scopes = ["userinfo-email", "compute-ro", "storage-ro"]
  }
```

The for_each meta argument will assign the values to the arguments from the list with key-value pair. While we can now test the above  template. For generalizing the network, subnet and load balancer related stuffs, I will post in the future articles.



`terraform apply -var-file=app-<env>.tfvars`



And the above command create compute instances depending on the .tfvars files passed while applying .

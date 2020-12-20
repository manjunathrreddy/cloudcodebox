---
title: Terraform And GCP Series - Part 1
date: 2020-10-04 21:59:58
tags: [Terraform, GCP]
---



This and the next series of posts will demonstrate the simplification of introducing complexity in IaC best practices. But first a simple Terraform script to provision resources on a GCP cloud. We dive into getting a VM instance with Apache web server with in Google Cloud Platform public in public cloud. We start with one main.tf which has all the configurations and the resources to provision and orchestrate in GCP.

Lets first define the provider configurations:

```Terraform
provider "google" {

project = "triple-virtue-271517"
version = "~> 3.38.0"
region = "us-central1"
zone = "us-central1-a"
credentials = "${file("${var.path}/cloud-access.json")}"

}
```

The **path** variable  refers to the access tokens  to GCP cloud project as below:

```
variable "path" { default = "/home/vagrant/gcp_credentials/keys" }
```



Lets define the firewall rules with default network resources

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

The **target_tags** defined shall then be referred in the resources (VM instances) that may require the firewall rules to enable http and https ports.

Further we will define code to provision a VM instance and map it to the default network with above mentioned firewall rule

```
resource "google_compute_instance" "test_instance" {
    
    name            = "demo-01"
    machine_type    = "e2-standard-2"
    zone            = "us-central1-a"
    metadata_startup_script = <<-EOF
    sudo yum update 
    sudo yum install httpd -y
    sudo systemctl start httpd
    sudo systemctl start httpd
    sudo systemctl enable httpd
    EOF

    tags = ["allow-http","allow-https"]

    boot_disk {
      
    initialize_params{

          image           = "centos-8"
          size            = "100"
            

        }
    }

    network_interface {
        network = "default"
    
            access_config {
      // Ephemeral IP
        }
    }

    service_account {
        scopes = ["userinfo-email", "compute-ro", "storage-ro"]
    }

}
```

The **metadata_startup_script** also tries to install webserver while provisioning the vm instance. The **network_interface** section assigns a public ip to the same instance.

Now putting all together 

**main.tf**

```
variable "path" {  default = "/home/vagrant/gcp_credentials/keys" }

provider "google" {
    project = "triple-virtue-271517"
    version = "~> 3.38.0"
    region = "us-central1"
    zone = "us-central1-a"
    credentials = "${file("${var.path}/cloud-access.json")}"
  
}

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


resource "google_compute_instance" "test_instance" {
    
    name            = "demo-01"
    machine_type    = "e2-standard-2"
    zone            = "us-central1-a"
    metadata_startup_script = <<-EOF
    sudo yum update 
    sudo yum install httpd -y
    sudo systemctl start httpd
    sudo systemctl start httpd
    sudo systemctl enable httpd
    EOF

    tags = ["allow-http","allow-https"]

    boot_disk {
      
    initialize_params{

          image           = "centos-8"
          size            = "100"
            

        }
    }

    network_interface {
        network = "default"
    
            access_config {
      // Ephemeral IP
        }
    }

    service_account {
        scopes = ["userinfo-email", "compute-ro", "storage-ro"]
    }

}

output "machine_type" {
  value = "${google_compute_instance.test_instance.machine_type}"
}

output "name" {
  value = "${google_compute_instance.test_instance.name}"
}

```



Here below are the results of the above resources created in GCP.

The server instance created in the vm console



{% asset_img gce_server_jpg.jpg %}



The Apache webserver running in that instance



{% asset_img apache_server_jpg.jpg %}



In the next part we will further refine the above script by splitting the script into different files and terraform interpolation.






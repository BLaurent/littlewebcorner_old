---
title: "VMs 30 Minutes"
date: 2018-05-14T11:18:00+02:00
draft: false
image: "img/portfolio/bike1.jpg"
tags:
- vagrant
- packer
- terrafrom
- ansible
categories:
- post
description: "A short tutorial to create to create and deploy VMs"
---


# Pre Requisite
I am going to focus on installing thing on OSX only. I assume if you are here you know enough to figure out how to do the same on your OS.


{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}

brew cask install virtualbox virtualbox-extension-pack doctl
vagrant plugin install vagrant-vbguest
brew install packer ansible terraform-provisioner-ansible terraform
brew cask install vagrant

{{< / highlight >}}


# Source code

All ressources listed in this tutorial are available in this repository :
[lighting_talk](https://github.com/blaurent/lighting_talk/) in the directory *virtualization*. Go to this directory.

# Vagrant
Its a very powerfull tool to help create a virtual machine from a simple description file. It's generic meaning that having one description allow you to create VM on many virtualization hypervisor.

The first VM we are going to create is a simple ubunutu. Go to the directory named *virtualization* and enter the following commands
{{< highlight Ruby "nobackground=true" >}}
vagrant up
{{< / highlight >}}

After few minutes you should be able to login to this VM using :

{{< highlight Ruby "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
vagrant ssh
{{< / highlight >}}

Open the Vagrantfile to check what kind of VM have been created.

{{< highlight Ruby "linenos=table,hl_lines=8 15-17,linenostart=1" >}}

Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox"
  config.vm.box = "ubuntu/xenial64"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false
    # Customize the amount of memory on the VM:
    vb.memory = "256"
    vb.cpus = "1"
  end
end
{{< / highlight >}}

You are running an Ubuntu xenial 64 bit with 256M of Ram and 1 cpu.

# Ansible
We could setup a VM by hand since we have ssh, a couple of *apt-get* should be enough to configure it. But What happen if you need to create 1000VM in the cloud with auto scaling, you are going to need a lot of chinese intern...
Having a simple VM configured by script is already something by we want to install more things on it. Ansible allow that to define a playbook, with a lot of rule to install and configure your VMS. Ansible can work on multiple Linux Flavor, be composed if you want to mix several components, enforce security,...

At the end of the Vagrantfile you should have something like that :

{{< highlight Ruby "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
config.vm.provision "ansible" do |ansible|
  ansible.playbook = "./ansible-nginx/playbook.yml"
  ansible.extra_vars = {
    ansible_python_interpreter: "/usr/bin/python3",
    }
end
{{< / highlight >}}


Uncomment it and run the following command

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
vagrant destroy -f && vagrant up
{{< / highlight >}}

# Packer

After creating our VM locally it time to generate images to deploy our application to several platform. For that we have packer. Packer allow you to build image for specific  format e.g AMI for AWS, OVF or ISO for virtualbox, Droplet for Digital Ocean,...

For now we will generate a Virtual box iso:

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
packer build -only=virtualbox-iso packer.json
{{< / highlight >}}

In the packer.json since the config is written with headless="true" you will not see the auto install.

After a while you will have several files that are ready to import in VirtualBox

## Launching our new image
You need to ensure that any VMs we may have launched previously is shutdown
{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
vagrant suspend
{{< / highlight >}}

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
cd duck
vagrant up
{{< / highlight >}}

After a while you will have a web server running in http://localhost:8888

## Create a new image for digital ocean

First things to do is to set the API key with yours. You can set the env variable *DIGITALOCEAN_API_TOKEN*
with you api key and everything should run smoothly.

{{< highlight json "linenos=table,hl_lines=8 15-17,linenostart=1" >}}

{
  "type": "digitalocean",
  "image": "ubuntu-16-04-x64",
  "region": "fra1",
  "size": "512mb",
  "ssh_username": "root"
},

{{< / highlight >}}


Then to create an image compatible with Digital Ocean, you just need to launch :
{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
    packer build -only=digitalocean packer.json
{{< / highlight >}}

In the end you will have a nice image reading to be used with a droplet

Get the list of image, region, machine available using those three simple commands:
{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
doctl compute image list
doctl compute region list
doctl compute size list
{{< / highlight >}}

Create a droplet using this image
{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
doctl compute droplet create --image XXXXXX --region fra1 --size s-1vcpu-1gb packer-test
{{< / highlight >}}

where *XXXXX* will be replace by the image id you have retrieve with the previous three commands.

After that you need to get the public IP of your droplet
{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
doctl compute droplet list
{{< / highlight >}}

Open a web browser and type the following URL: http://IP
Et Voila !!!

Do not forget to destroy all images and droplets you have created otherwise you will be charged !!
To get ids of all ressources allocated:

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
doctl compute image list
doctl compute droplet list
{{< / highlight >}}

And to Clean up pass those ids along

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
doctl compute image delete XXX
doctl compute droplet delete YYYY
{{< / highlight >}}


# Terraform

We done the deployement to Digital Ocean by hand now we are going to use terraform template to create a droplet with the image we have created previously.
We also need to set the API key to avoid hard coding it to the *.tf* file. Set *DIGITALOCEAN_TOKEN* variable with your Digital Ocean API key.

Start by validating the setup of your configuration.

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
terraform validate
{{< / highlight >}}

You might not have all provider plugin installed in your machine, if so launch :

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
terraform init
{{< / highlight >}}

You are now ready to *apply* your configuration.

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
terraform apply
{{< / highlight >}}

After that you need to get the public IP of your droplet
{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
doctl compute droplet list
{{< / highlight >}}

Open a web browser and type the following URL: http://IP
Et Voila !!!

We could have created a dns entry but since propagation is a bit slow for the sake of the exercise we keep it simple using IP only.

Do not forget to destroy all droplet using this command if you do not want to be charged.

{{< highlight bash "linenos=table,hl_lines=8 15-17,linenostart=1" >}}
terraform destroy
{{< / highlight >}}

# Conclusion

We have seen rapidly that creating a custom VM image is not that difficult and deploying to the cloud this exact same VM neither.
For this exercise we've been using Digital Ocean (Thanks for the free credits by the way). But we could have done it with Azure, AWS or GCP.
Integrating this into a CI/CD pipeline still require some work (not much) and the Ansible part need some refinement to comply with the way ansible galaxy works.
I know it's an old subject first article 2014/2015, but it's always nice to review old/new tools sometimes.

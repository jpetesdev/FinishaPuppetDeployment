# FinishaPuppetDeployment
Week 2 Task from Google IT Automation Configuration Management

Introduction

You need to automatically manage the computers in your company's fleet, including a bunch of different machines with different operating systems. You've decided to use Puppet to automate the configurations on these machines. Part of the setup is already done, but there are more rules that need to be added, and more operating systems to consider.
Prerequisites

Understands the basics of Puppet, including:

    How to create Puppet classes and rules

    How Puppet interacts with different OSs

    How to use the DSL to create complex rules


Puppet rules

The goal of this exercise is for you to see what Puppet looks like in action. During this lab, you'll be connecting to two different VMs. The VM named puppet is the Puppet Master that has the Puppet rules that you'll need to edit. The VM named linux-instance is a client VM that you'll use to test that your catalog was applied successfully.

The manifests used for the production environment are located in the directory /etc/puppet/code/environments/production/manifests, which contains a site.pp file with the node definitions that will be used for this deployment. On top of that, the modules directory contains a bunch of modules that are already in use. You'll be extending the code of this deployment to add more functionality to it.

Task 1:
add an additional resource in the same init.pp file within the path /etc/puppet/code/environments/production/modules/packages, ensuring the golang package gets installed on all machines that belong to the Debian family of operating systems (which includes Debian, Ubuntu, LinuxMint, and a bunch of others).

class packages {

    package { 'python-requests':
        ensure => installed,
    }

    if $facts[os][family] == "Debian" {
      package { 'golang':
        ensure => installed,
      }
    }

}


After this, we will also need to ensure that the nodejs package is installed on machines that belong to the RedHat family. Refer to the below snippet for this.

Need to check if config works correctly. 
How to fetch IP address of another machine on GCLOUD
gcloud compute instances describe linux-instance --zone=us-central1-a --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
34.69.83.64


Once connected to the linux machine we need to check if our rules work correctly.

sudo puppet agent -v --test



Fetch machine information

It's now time to navigate to the machine_info module in our Puppet environment. In the Puppet VM terminal, navigate to the module using the following command:

cd /etc/puppet/code/environments/production/modules/machine_info

The machine_info module gathers some information from the machine using Puppet facts and then stores it in a file. Currently, the module is always storing this information in /tmp/machine_info.

student-00-1bf6b559fa69@puppet:/etc/puppet/code/environments/production/modules/machine_info$ cat manifests/init.pp 
class machine_info {

    file { '/tmp/machine_info.txt':
        content => template('machine_info/info.erb'),
    }


}



Reboot machine

For the last exercise, we will be creating a new module named reboot, that checks if a node has been online for more than 30 days. If so, then reboot the computer.

To do that, you'll start by creating the module directory.

Switch back to puppet VM terminal and run the following command:

sudo mkdir -p /etc/puppet/code/environments/production/modules/reboot/manifests

Create init.pp file and edit

In this file, you'll start by creating a class called reboot.

The way to reboot a computer depends on the OS that it's running. So, you'll set a variable that has one of the following reboot commands, based on the kernel fact:

    shutdown /r on windows
    shutdown -r now on Darwin (macOS)
    reboot on Linux.

Hence, add the following snippet in the file init.pp:

class reboot {
  if $facts[kernel] == "windows" {
    $cmd = "shutdown /r"
  } elsif $facts[kernel] == "Darwin" {
    $cmd = "shutdown -r now"
  } else {
    $cmd = "reboot"
  }
  if $facts[uptime_days] > 30 {
    exec { 'reboot':
      command => $cmd,
     }
   }
}

To make sure the code gets run you need to add it to the sites.pp file
node default {
    class { 'packages': }
    class { 'machine_info': }
    class { 'reboot': }
}



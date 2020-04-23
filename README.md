# Kubernetes-Cluster-Ubuntu-18.04

### Objective

* The objective is to install Kubernetes on Ubuntu 18.04 Bionic Beaver Linux

### Operating System and Software Versions
   
* Operating System: - Ubuntu 18.04 Bionic Beaver Linux
   
   * Software: - Kubernetes v1.12.0

### Requirements

* Privileged access to your Ubuntu System as root or via sudo command is required.

* VirtualBox or another virtualization solution installed

   * All steps can be executed in LInux, Windows and/or macOS environment as well.
   
__Download links:__

   [Vagrant Download](https://www.vagrantup.com/downloads.html)
   
   [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

   [Vmware](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html)
   
   
* Deploy 3 nodes cluster with Vagrant
   
    * Initiate the creation of the cluster:
            
          time vagrant up
   
   * Clean up everything by executing:
      
         vagrant destroy --force

### Verify installation

* Log in to master: 
    
      vagrant ssh k8s1

* Now let’s see some information about the cluster:
      
      kubectl cluster-info
 
* We can check the nodes in the cluster:
     
      kubectl get nodes

* Then get some information about the running pods:

      kubectl get pods

* And finally, list all pods including the system ones:

      kubectl get pods --all-namespaces

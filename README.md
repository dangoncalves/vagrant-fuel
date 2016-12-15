## Deploy Fuel 9 with Vagrant libvirt provider
### Prepare environment (Ubuntu 16.04)

* Install dependencies

         $ sudo apt-get install git qemu libvirt-bin libvirt-dev zlib1g-dev ruby-dev bundler

* Install Vagrant (latest version on [Download page](https://www.vagrantup.com/downloads.html))

         $ wget https://releases.hashicorp.com/vagrant/1.9.1/vagrant_1.9.1_x86_64.deb
         $ sudo dpkg -i vagrant_1.9.1_x86_64.deb

* Clone and install vagrant-libvrit provider

         $ git clone https://github.com/vagrant-libvirt/vagrant-libvirt.git
         $ cd vagrant-libvirt
         $ rake build
         $ vagrant plugin install ./pkg/vagrant-libvirt-*.gem

### Provide deployment configuration

Clone this repository:

    $ git clone https://github.com/michalskalski/vagrant-fuel
    
`Vagrantfile` include definition of Fuel Master VM and slave nodes.
`env.conf.sample` contains example of environment configuration, it should be adjusted before deployment:

    $ cp env.conf.sample env.conf
    $ source env.conf
    
Remember to source `env.conf` file before executing any vagrant VMs related commands.
    
### Deployment

[Centos 7.2 box](https://atlas.hashicorp.com/centos/boxes/7/versions/1610.01) is used as base for Fuel Master node and on the top of this Fuel packages are installed. Slave nodes will be booted from network.
There are two scenarios of deployment depend on value of `VIRTUAL` env variable:

1. If set to 'true' it is assumed that all nodes in cluster will be local VMs and for communication between them libvirt networks will be used.
2. If set to 'false' existing bridges/interfaces defined in `env.conf` file will be used so it should be possible to have both virtual and physical nodes for ex. Controllers in VMs and Computes on physical servers

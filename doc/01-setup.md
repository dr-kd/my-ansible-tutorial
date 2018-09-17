# Initial set up

This mini tutorial is written for me from my current point of view.  That is, I have some limited experience working with other people's ansible playbooks, but I haven't built my own from scratch.

Ansible is generally for multi-machine interactions.  So we'll start by setting up a three machine vagrant cluster.

## Vagrant

We'll start with three machines running with vagrant (see the Vagrantfile in the repo root).

### Initial setup:

* Download the base VM image:

    vagrant box add debian/stretch64

* Start the VMs

    vagrant up

* Grab the vagrant ssh config to a file:

    vagrant ssh-config > vagrant_ssh_config


* Test we can ssh to each:

    for i in {1..3}; \
        do ssh -F vagrant_ssh_config box${i} echo hi there box ${i}; \
    done




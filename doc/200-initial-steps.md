# Initial steps with ansible

Let's reproduce the first piece of work except via ansible.

## Hosts and group vars.

First up, check the file `myhosts` and the file `group_vars/mycluster` for setup.  Note that the machines are overall known as `mycluster`. They contain box1 referred to as `main_box`, with `box2` and `box3` as `other_boxes`.

The `group_vars/mycluster` file lets ansible know how to ssh to the three hosts.

## Running the example command via `ansible`

    ansible  -i ./myhosts mycluster -a 'echo hi there {{ansible_host}}'

At which point you'll see three lots of output like this:

    box1 | SUCCESS | rc=0 >>
    hi from box1

Note that if you run it repeatedly the ordering will be different as ansible executes by default in parallel.  To run one at a time add `-f 1` (fork one process at a time):

    ansible  -f 1 -i ./myhosts mycluster -a 'echo hi there {{ansible_host}}'

## Making the hosts file more permanent

Finally we can add a line defining the hosts to `ansible.cfg` so that ansible will use those hosts by default.

    [defaults]
    inventory=./myhosts

Which now means we can run:

    ansible mycluster -a 'echo hi from {{ansible_host}}'

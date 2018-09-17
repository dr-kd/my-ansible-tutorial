# Starting with ansible roles.

While having a single task file to group tasks together is useful, it is often preferable to use roles to split common behaviour up into components.

We're going to take the functionality we created before and compose it into roles.

Ansible roles contain a fair bit of boilerplate, so it's useful to have code to kick them off for you.  The command `ansible-galaxy` does this.

So to start we make stubs for the new roles:

    ansible-galaxy  init roles/say_hello
    ansible-galaxy  init roles/install_fortune
    ansible-galaxy  init roles/dynamic_motd
    ansible-galaxy  init roles/static_motd

## Porting say_hello to a role.

For the first task in the `main.yml` playbook, we can easily change that to be a role.  Replace the first job (on hosts mycluster) replace with the following:

    - hosts: mycluster
      roles:
        - say_hello

And in `roles/say_hello/tasks/main.yml` use this as the file contents:

    ---
        - name: say hi
          shell: echo hi there {{ ansible_host }}

The ansible playbook will run unchanged

## Porting remaining actions to a role.

We can use the same process to port `install_fortune` and `dynamic_motd`.  Although these are being implemented as two separate roles, it might be a better design to have them as a single role.

Finally we can port over the `static_motd` behaviour in the same manner.  This leaves us with the following `main.yml` file:

    ---
    - hosts: mycluster
      name: just a test to say hi
      roles:
        - say_hello
    - hosts: main_box
      name: install fortune and update motd
      roles:
        - dynamic_motd
    - hosts: other_boxes
      name: static motd on other boxes
      roles:
        - static_motd

Which is simpler, easier to follow, and separates the description (what we're trying to achieve) from the implementation detail.

Note that our use of ansible-galaxy created a bunch of other files used in various circumstances.  Roles are quite well documented in the ansible documemntation, so I won't go through that here.

## A note on dependencies.

Dependencies are interesting and useful, but are a little dangerous.  The reason for this is that you could accidentally introduce circular dependencies

So for the following example we could remove the instll_fortune role from `main.yml` and instead add it to `roles/dynamic_motd/meta/main.yml` in the dependencies section.  Try this out by issuing the command `vagrant destroy` to blow away your virtual machines, and then reinstantiate them with `vagrant up`.

After this confirm that the motd in each box is back to default by issuing the command `vagrant ssh box1` and so on.

Finally, run `ansible-playbook main.yml` and confirm all the behaviour we configured previously has returned.



# Adding another task to the playbook

Ansible has lots of modules.  For example the `apt` module manages debian
packages.  These are generally well documented, and pretty easy to follow.

## Adding a debian package to one group of machines

Let's install the debian package `fortune-mod` on the `main_box` by adding the following to `main.yml`

    - hosts: main_box
      name: install fortune
      tasks:
      - name: install fortune-mod
        become: yes
        apt:
          name: fortune-mod
          state: present

Running the playbook again `ansible-playbook  main.yml` will now install fortune-mod on `box1` but not `box2` or `box3`.  Running the playbook a second time will output as follows (at the end):

    box1 | SUCCESS => {
    "cache_update_time": 0,
    "cache_updated": false,
    "changed": false
    }

This indicates that the `fortune-mod` was already installed and no action was necessary.

## Updating message of the day to use the debian package

Next we'll extend that group of tasks to use `fortune` to provide the login message of the day (motd).  First up, because this uses a shell command we need to check the state ourself to see if performing the task is necessary:

       - name: check motd not already updated
         stat: path=/etc/profile.d/motd.sh
         register: motd

Which means we register the variable `motd` to the playbook for future use.

Next we update the script that provides the motd, if necessary:

       - name: update motd
         become: yes
         shell: echo /usr/games/fortune > /etc/profile.d/motd.sh
         when: motd.stat.exists == False

The `when` directive says to only run when the file does not exist.

## Updating message of the day on the other machines

Finally we'll make it so that the remaining machines print out their `ansible_hostname` in the motd on login.

First the boilerplate indicating where to run:

    - hosts: other_boxes
      name: static motd on other boxes
      tasks:

Then we need to check if the file `/etc/motd` is changed from its default or not.

      - name: check motd file is not modified
        shell: grep -q {{ ansible_host }} && echo changed || echo unchanged
        register: static_motd

Note here we use the shell command to output 'changed' if it contains the ansible_host name, or 'unchanged' if it doesn't.  This kind of thing can be fragile, so it pays to treate these conditionals with care, and use for-purpose ansible modules (like `apt` or `stat`) where possible.

Finally we modify `/etc/motd` if the `static_motd` variable has the contents "unchanged".

    - name: create motd
        become: yes
        shell: echo Hi there, welcome to {{ ansible_host }} > /etc/motd
        when: static_motd.stdout == 'unchanged'

After running all this, you can verify that `vagrant ssh box1` and `vagrant ssh box2` results in the expected output.

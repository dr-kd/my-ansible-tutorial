# Starting with an ansible playbook

The `ansible` command is useful for running arbitrary commands on hosts.  For example I've used it for assessing clock drift across a cluster: `ansible mycluster -a 'date -u'`.  But really ansible wants you to use playbooks via the 'ansible-playbook' command.

We'll reproduce the intial test setup again, this time with 

## Initial setup

We'll create a file called `main.yml` in the root directory.

    ---
    - hosts: mycluster
      tasks:
      - name: say hi
        command: echo hi there {{ ansible_host }}

And then we can run it:

    ansible-playbook -v main.yml

We use the `-v` script here because the output of successful commands will not echo unless ansible is in verbose mode.  Add more `v`s to the command for increasing verbosity.

To demonstrate that unsuccessful exit status causes verbosity we can replace the `command` part of the playbook task with this:

    shell: echo hi there {{ ansible_host }} && false

Note the use of the `shell` module rather than the `command` module.  You'll want this whenever you need to use pipes, or other shell constructs.

This time running `ansible-playbook main.yml` will indicate the commands "failed" and their output will be visible.

## A note on the output of running ansible_playbook

You can make the ansible output more readable by placing the following into ansible.cfg

    stdout_callback=minimal


Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m79[39m }
Unknown markup type 10 {
  type: [33m10[39m,
  start: [33m121[39m,
  end: [33m126[39m,
  href: [32m''[39m,
  title: [32m''[39m,
  rel: [32m''[39m,
  name: [32m''[39m,
  anchorType: [33m0[39m,
  creatorIds: [],
  userId: [32m''[39m
}
Unknown markup type 10 { type: [33m10[39m, start: [33m176[39m, end: [33m196[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m247[39m, end: [33m253[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m96[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m177[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m307[39m, end: [33m312[39m }

# Ansible custom facts



Whenever you run an Ansible playbook, the first thing that happens is the setup task. This task gathers a whole host of facts about the remote machine(s) — IP addresses, disks, OS version… you name it. Ansible refers to these as “facts”.

But, did you know you can also add your own custom facts that Ansible can go and fetch for you?

I’ve never been able to find any proper documentation for this feature, but it is at least hinted at in the [docs ](http://docs.ansible.com/ansible/setup_module.html)for the setupmodule. This is a module that Ansible runs implicitly at the start of any playbook run, and is used for gathering facts.

Despite the lack of documentation, custom facts are actually quite easy to implement.

On any Ansible controlled *host* — that is, the remote machine that is being controlled and not the machine on which the playbook is run — you just need to create a directory at /etc/ansible/facts.d. Inside this directory, you can place one or more *.fact files. These are files that return JSON data, which will then be included in the raft of facts that Ansible gathers at the start of each playbook run.

You can name them anything you like, as long as they use the *.fact extension. Here’s a somewhat redundant example that just tells us the date and time —

    #!/bin/bash
    DATE=`date`
    echo "{\"date\" : \"${DATE}\"}"

If you include *.fact files that are executable (like the one above) then Ansible will run them and expect JSON on stdout. If you include files that are *not* executable and simply contain raw JSON then Ansible will just read them and use the data inside.

Once you’ve got these files in place on your remote machines, they can be accessed in a playbook at

    hostvars.host.ansible_local.*

…where * is the name you used for the .fact file. E.g. date_and_time.fact would be available at hostvars.host.ansible_local.date_and_time.

### Setting up custom facts automatically

You probably don’t want to manually place your *.fact files on each remote machine. Since we’re using Ansible anyway, it makes sense to create a playbook that creates our *.fact files for us.

    - name: "Create custom fact directory"
      file:
        path: "/etc/ansible/facts.d"
        state: "directory"

    - name: "Insert custom fact file"
      copy:
        src: files/custom.fact
        dest: /etc/ansible/facts.d/custom.fact
        mode: 0755

This assumes that you have a script at files/custom.fact that will return some JSON.

It’s worth noting, however, that the above playbook will only *install* the custom fact — it won’t actually execute it because custom facts are loaded by the setup module and that only runs at the *start* of a playbook. If you want to install a custom fact and use it in the same playbook you’ll need to re-run setup —

    - name: "Re-run setup to use custom facts"
      setup: ~

That’s all there is to it — now you can make use of custom facts in your playbooks.

Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m161[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m222[39m, end: [33m226[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m117[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m173[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m124[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m138[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m151[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m200[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m186[39m, end: [33m196[39m }

# Scrape AWS Public IP Json + Ansible + AWS Firewall



Every data center I’ve ever worked at has had some sort of egress internet filtering for security. This makes perfect sense in a world where partners have static IPs, or even when partners can provide the range of IPs they have registered with IANA. But in the cloud world? Not so much.

For those that aren’t familiar, most cloud providers today don’t permit spinning up hosts and using an inclusive range of IPs owned by the client. Rather, they use IPs they own, which are random and never conjoined. Not to mention, they are provisioned dynamically, so if a host or load balancer is rebuilt (more and more common in a DevOps world), it will come back up with a new public IP.

Clearly, static IP whitelisting isn’t a good solution. However, whitelisting the entire internet isn’t a great solution either. I recently received an assignment at work:
> Whitelist our data centers to AWS

On paper, it’s so simple. However, we had additional requirements:
> Only whitelist the IPs for Amazon prime services — not those owned by clients

and
> Whitelist only the us-east-1 region IPs

Now, Amazon controls a great deal of the internet. They are constantly acquiring and deploying new ranges, and even worse, splitting up previously assigned ranges and assigning them to different services. To help customers cope, they publish a .json with all the IPs they control, as well as the associated services (ec2, Cloudfront, Amazon-owned infrastructure) and the region it operates out of (us-east-1, us-west-1, etc.). The file looks like this:

<iframe src="https://medium.com/media/868e776d6e11bb95a8ca76995a1c5842" frameborder=0></iframe>

Except continue that pattern for a tidy 12k+ lines. Auditing it by hand would hardly be doable, much less if the things change every 6 hours and I’d need to diff it and adjust our firewalls accordingly.

## So in Walks Ansible

Ansible is an automation tool that is capable of SSH’ing into hosts and making changes. Passwords can be stored in a secure file outside of source control, and it could conceivably handle this job, provided I could provide it a list of addresses to the whitelist.

It’s one I hadn’t used before, so it seemed like a fun opportunity to take it for a spin.

Ansible operates on the idea of playbooks that contain instructions for what commands to send and to whom. It’s dynamic and flexible, but not flexible enough to read a list of networks and push them to object-groups on a firewall.

My first and favorite programming language is bash (hey, it’s simple!) so I decided to create a preprocessor in bash to build an ansible playbook based on the list of networks we want to whitelist.

## So here’s the plan

So that’s the plan:

1. Download the .json that Amazon publishes

1. Filter it and select only the networks that we need

1. Format the list of subnets into a list that bash can consume

1. Have bash iterate over the list and build an ansible-playbook

1. Use Cron to have the script run every few minutes

## A Wrinkle

I quickly realized that Ansible doesn’t have a state of what networks are “new” and should be added, or are “different” and should be changed, or “removed” and should be… well, removed. It’s possible to do some diffing but it sounded like a time-suck, and I wanted to build this quickly. So I came up with a terrible idea (but it works though!).

We can create two object-groups and whitelist to both of them. Only the first one gets hits. However, when we recognize that the list of IPs has changed at ALL (a much easier task than keeping track of WHAT changed), we can remove the first object group (traffic is still whitelisted by the second), rebuild the first object-group (traffic is now whitelisted by the first), then rebuild the second object-group (to prep for next time). With that fancy tango, we have a dynamic DevOpsy house of cards all glued together. But again, IT WORKS.

## Step 1: Download the .json that Amazon publishes

First, let’s download the JSON to a Linux host. We also set a DATE variable so we can write it to the object-group so it’s easy to tell later when it was last updated. We cd to the directory we’d like to execute from, and wget our JSON file.

Note the -N flag on wget, it’s important. It means that wget will check if the file differs in any way. If it doesn’t, it won’t download the file.

<iframe src="https://medium.com/media/108c6d87ccab83da4805c2d1a6a52b1e" frameborder=0></iframe>

Since this script will run frequently (I run it every 3 minutes, but you could run it every 30 seconds if you wanted), and we probably don’t want to just hammer the firewall with a constant churn of object-group updates, we need a test to tell if our existing config is still valid. The easiest way to do that is to check the date-stamp — if wget downloaded a new file, it has changed, and let’s proceed. If not, we write a log entry and exit.

<iframe src="https://medium.com/media/4812f7254fe9b0a8d75d94ad6cffc227" frameborder=0></iframe>

This one line is a doozy, so let’s walk through all the cool stuff it does. First, we execute jq, or javascript query, and tell it to output raw strings, rather than json -r, and then read the ip-ranges.json file and filter it for region us-east-1 and service AMAZON.

<iframe src="https://medium.com/media/8ae6f452e693b3f0d92b0efe07f27df5" frameborder=0></iframe>

The output looks something like this:

<iframe src="https://medium.com/media/17f9960413272c59ef8f6410479aaab9" frameborder=0></iframe>

Now, the ASA can’t understand a list like that. Its syntax looks more like this: 52.72.0.0 255.254.0.0. But let’s not worry about that yet. First, we remove the old bash-built playbook to make way to build our new one and then we start writing static stuff — items that will be the same on each run.

We use a [heredoc](https://en.wikipedia.org/wiki/Here_document) to write a big block of text without needing to echo append every single line into a file.

<iframe src="https://medium.com/media/6932874428b3659b9a9c81fa8ec60a32" frameborder=0></iframe>

Now it’s time for the dynamic part, where we ingest the list of CIDRs we created earlier, format them for the firewall, and build them into an Ansible playbook. It’s worth going through this one deeply too — it’s the really cool stuff.

First, we do a while loop to read a document — our CIDRlist. For each line, we filter the input and assign a value as subnet and value as netmask. The netmask variable uses a common Linux tool called ipcalc that can convert CIDR slash notation (e.g., 10.0.0.0/8) to a blown-up citation (e.g. 10.0.0.0 255.0.0.0).

With those values in hand, we echo append the proper spacing (important for YAML docs), along with the Ansible playbook syntax, and insert our subnet and netmask values for this loop. Then we loop again and again… through every single CIDR we’ve downloaded.

<iframe src="https://medium.com/media/e212445c06ae867c977c71d3fd3b7ced" frameborder=0></iframe>

We have to do that twice — 1 for the object-groupA and one for object-groupB. Remember, no downtime is good downtime. Then we hand-off our generated playbook to ansible to do the heavy lifting, and echo a finish timestamp to our log.

<iframe src="https://medium.com/media/c232258f5d02a852e3de1486b51be8f3" frameborder=0></iframe>

And don’t forget to tell your host to run this bash generator every 3 minutes (or however quickly you’d like). It’s a good idea to use a service account (rather than your own user). Run crontab -e to jump into editing the cron file, which will execute commands at specific intervals and times and add this line:

<iframe src="https://medium.com/media/7fda27527267fc43150455c3cf189e3d" frameborder=0></iframe>

The source code for the bash ansible-playbook generator, as well as an example generated playbook, are here:
[**KyMidd/AnsibleAwsReader_PushToCiscoASA**
*You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…*github.com](https://github.com/KyMidd/AnsibleAwsReader_PushToCiscoASA)

I hope you enjoyed it, and good luck out there! 
kyler

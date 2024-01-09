# Proxmox Ansible Vm Creation Automation

This playbook automates creation of Debian 11 or 12 virtual machines on Proxmox (test on versions 7.x & 8.1.3).

Testing with Ubuntu 22 has fail thus far, but I'll keep at it.  

## Vaults

## step 1:

### linux user creation

In preperation for Ansible API usage, let's create a user

the `pve_onboard.yml` will create the ansible user with sudo rights, install sudo, and pass the ssh pub key you created

`ansible-playbook -i pvenodes 1.pve_onboard.yml`

Next, we'll install proxmoxer to your Proxmox host

`ansible-playbook -i pvenodes 2.pve_install_proxmoxer.yml`

### Proxmox user creation

Now although we can login to the operating system using SSH, I want to manage PVE itself using the REST like API that Proxmox has provided

To do that we’ll create a user account and API token

So in the GUI, navigate to Datacentre | Permissions | Users

Click Add, fill in the details and then click Add

At the very least we need to provide the User name and this needs to match the user account we just created

This is because we’re using Linux for the user database, and hence why we aren’t asked for a password here

Next we need to assign permissions for this user account

Navigate to Datacentre | Permissions and from the Add drop-down menu select User Permission

For the Path select / i.e. the root folder

For the user, select the one for Ansible

For the Role select Administrator

Now click Add

### API Usage

To setup an API token, navigate to Datacentre | Permissions | API Tokens

Click Add and select the Ansible user from the drop down menu

Enter a Token ID e.g. ansible-token

To avoid complications de-select the Privilege Separation option, otherwise we’ll have to handle different privileges for the user account and token

EXAMPLE:

```
Token ID
ansible@pam!ansible-token
Secret
848d5601-a186-4532-9199-bb96e7f33bfa
```


Enter this information in the group_vars/pvenodes/vault file with the command `ansible-vault edit group_vars/pvenodes/vault`

## step 2

create vaults

Refer to API USAGE above for instruction on how to create user and API token

`ansible-vault create group_vars/pvenodes/vault`

set password as needed

follow example for entries needed

Do the same for `group_vars/all/vault`

## step 3. (optional)

create password vault file so you don't have to type the vault pass everytime

`echo <password to vault(s) >> ~/.vaultpassfile.txt`

then you can run playbooks like this:
`ansible-playbook -i pvenodes site.yml --vault-password-file=~/.vaultpassfile.txt` and not have to type the password

Use a . in the filename to make it a hidden file in Linux

# SSH key

You'll need 2 keys.  One for Ansible access, and another for user acces as it's best practice for role seperation

I suggest using an updated key algorithm with this command (but you do you, boo):

`ssh-keygen -t ed25519`

I created `ansible_id_ed25519` for Ansible usage and `personal_id_ed25519` for my user access

# Var file update

Update `group_vars/all/vars` file with your applicable information.  Do the same with `group_vars/pvenodes/vars`

## all vars
update ansible_key_file, locale, and timezone at the minimum

## pvenodes vars
drive_storage and linux_bridge at the minimum require updating 

## group_vars/pvenodes/vms
Edit group_vars/pvenodes/example_vms (both static and dhcp are used as examples in there), save as group_vars/pvenodes/vms

## inventory file update
Copy sample-pvenodes to pvenodes
edit hostname/ip as necessary

## vm creation files
create vm files in variable_files/vms

Using examples/demo1 or examples/demo2 files as reference, create vms with applicable information for the vms you want to create.

# VM creation

You should be Good 2 Go

run with command `ansible-playbook -i pvenodes site.yml --ask-vault-pass`

# Cleanup post install

To cleanup a few things, like removing the cloud-init drive, post install clean reboot, etc.  

`ansible-playbook -i pvenodes 4.post_cleanup.yml --ask-vault-pass` 

or if you set up the vault pass file

`ansible-playbook -i pvenodes 4.post_cleanup.yml --vault-password-file=~/.vaultpassfile.txt`


# Credits
Huge shout out to David McKone at Tech Tutorials for setting the ground work.  

https://www.techtutorials.tv/sections/promox/

https://www.youtube.com/c/TechTutorialsDavidMcKone

## Vaults

### step 1:

create vaults
Refer to `API USAGE` below for instruction on how to create user and API token

`ansible-vault create group_vars/pvenodes/vault`

set password

follow example for entries needed

Do the same for `group_vars/all/vault`

## SSH key

You'll need 2 keys.  One for Ansible access, and another for user acces

I suggest using an updated key algorithm with this command (but you do you, boo):

`ssh-keygen -t ed25519`

I created `ansible_id_ed25519` for Ansible usage and `personal_id_ed25519` for my user access

## Var file update

Update `group_vars/all/vars` file with your applicable information.  Do the same with `group_vars/pvenodes/vars`

### all vars
update ansible_key_file, locale, and timezone at the minimum

### pvenodes vars
drive_storage and linux_bridge at the minimum require updating 

### group_vars/pvenodes/vms
Edit group_vars/pvenodes/example_vms (both static and dhcp are used as examples in there), save as group_vars/pvenodes/vms

## API usage

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

To setup an API token, navigate to Datacentre | Permissions | API Tokens

Click Add and select the Ansible user from the drop down menu

Enter a Token ID e.g. ansible-token

To avoid complications de-select the Privilege Separation option, otherwise we’ll have to handle different privileges for the user account and token

EXAMPLE:

`Token ID
ansible@pam!ansible-token
Secret
848d5601-a186-4532-9199-bb96e7f33bfa`

Enter this information in the group_vars/pvenodes/vault file with the command `ansible-vault edit group_vars/pvenodes/vault`

## VM creation

You should be Good 2 Go

run with command `ansible-playbook -i pvenodes site.yml --ask-vault-pass`
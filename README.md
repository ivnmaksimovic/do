# Server setup
- Create a Droplet (VM) in Digital Ocean
- Create Domains and DNS Records
- Do Initial Server Setup
- Setup Caddy Server
- Deliver Static Sites to Server
- Deploy Sites

## Install roles locally (dependencies)
This is only needed because some playbooks are dependant on some roles. Like caddy.yml.
`ansible-galaxy role install -r ansible/roles/requirements.yml -p ansible/roles`

## Setup inventory
Copy `inventory.template` to `inventory`.
For Digital Ocean copy Ansible Inventory DO from Bitwarden.

## Run playbooks
Inventory is set in `ansible.cfg`.

Examples use `bob` as an example ssh user with permissions to hosts.

Some playbooks require `export DO_API_TOKEN=your_do_api_token`. Copy from Bitwarden.

### Create a Droplet (VM) in Digital Ocean
TODO

### Create Domains and DNS Records
- Setup DNS records like CNAME for www
`ansible-playbook ./ansible/playbooks/cname-record.yml -e "site_name=adamontenegro.com" --user bob --ask-become-pass`

### Do Initial Server Setup
- Setup user. Add it to sudo and www-data groups. Get ssh key from root. This is first and only task that needs to be run as `root`.
  `ansible-playbook ansible/playbooks/user-setup.yml -e "user_name=bob" --user root`
- Ping to test user permissions. Example of passing inventory.
  `ansible -i ./ansible/inventories/inventory production -m ping --user bob`
- Enable firewall.
  `ansible-playbook ./ansible/playbooks/firewall.yml --user bob --ask-become-pass`
- Set timezone.
  `ansible-playbook ./ansible/playbooks/timezone.yml --user bob --ask-become-pass`
- Maybe update packages.
  `ansible-playbook ./ansible/playbooks/apt.yml --user bob --ask-become-pass`
- Restart.
  `ansible-playbook ./ansible/playbooks/restart.yml --user bob --ask-become-pass`

### Setup Caddy Server
- Install Caddy.
  `ansible-playbook ansible/playbooks/caddy.yml -e "email=me@example.com" --user bob --ask-become-pass`
- Uninstall Caddy.
    `ansible-playbook ./ansible/playbooks/caddy-uninstall.yml --user bob --ask-become-pass`
- Reload Caddy. Requires `DO_API_TOKEN` to be set.
    `ansible-playbook ./ansible/playbooks/caddy-reload.yml --user bob --ask-become-pass`

### Deliver Static Sites to Server
- Build
  `ansible-playbook ./ansible/playbooks/build.yml -e "site_name=adamontenegro.com" --user bob --ask-become-pass`
- Deliver site `dist` folder to server.
  `ansible-playbook ./ansible/playbooks/deliver.yml -e "site_name=adamontenegro.com site_version=v1.9.0" --user bob --ask-become-pass`
- Deliver site `dist` folder to server. Site name is not same as project folder name.
  `ansible-playbook ./ansible/playbooks/deliver.yml -e "site_name=adamontenegro.com project_name=ada-project-folder site_version=v1.9.0" --user bob --ask-become-pass`

### Deploy Sites
- Deploy a version.
  `ansible-playbook ./ansible/playbooks/deploy.yml -e "site_name=adamontenegro.com site_version=v1.9.0" --user bob --ask-become-pass`
- Read deployment log.
  `ansible-playbook ./ansible/playbooks/deployment-log.yml -e "site_name=adamontenegro.com limit=5" --user bob --ask-become-pass`

## Tips
To see all the magic variables
`ansible <hostname> -m ansible.builtin.setup --user root`
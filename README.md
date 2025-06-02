# DO

## Requirements
- [UV](https://docs.astral.sh/uv/getting-started/installation/)
  `curl -LsSf https://astral.sh/uv/0.7.8/install.sh | sh`

## Getting Started
- Sync the project's dependencies with the environment.
  `uv sync`
  This includes Ansible as well.
- Copy `inventories/inventory.template` to `inventories-prod` etc.
- Populate inventory files.
- Install roles locally
  `uv run ansible-galaxy role install -r roles/requirements.yml -p ./roles`
  Most playbooks don't need this.
- Run Ansible
  `uv run ansible -i inventories/inventory-lab lab -m ping --user root`

## Server setup
- Create a Droplet (VM) in Digital Ocean
- Do Initial Server Setup
- Create Domains and DNS Records
- Setup Caddy Server
- Deliver Static Sites to Server
- Deploy Sites

## Setup inventory
Copy `inventory.template` to `inventory`.
For Digital Ocean copy Ansible Inventory DO from Bitwarden.

## Run playbooks
Inventory is set in `ansible.cfg`.

Examples use `bob` as an example ssh user with permissions to hosts.

Some playbooks require `export DO_API_TOKEN=your_do_api_token`. Copy from Bitwarden.

### Create a Droplet (VM) in Digital Ocean
These playbooks use Digital Ocean API and not ssh.

- Set DO_SSH_KEY_FINGERPRINTS env var.
  Find your SSH key fingerprints in the DigitalOcean dashboard under "Security" > "SSH Keys" or via the DigitalOcean API
    `export DO_SSH_KEY_FINGERPRINTS="3b:16:...,another:fingerprint"`

- Create a Droplet
  `ansible-playbook playbooks/droplet-create.yml -e "droplet_name=my-droplet-name"`

- Destroy a Droplet
  `ansible-playbook playbooks/droplet-destroy.yml -e "droplet_name=my-droplet-name"`

### Do Initial Server Setup
- Setup user. Add it to sudo and www-data groups. Get ssh key from root. This is first and only task that needs to be run as `root`. This user will need sudo password so it will be prompted for it. This is dependant on `passlib`. 
  `ansible-playbook playbooks/user-setup.yml -e "user_name=bob" --user root`

- Ping to test user permissions. Example of passing inventory.
  `ansible -i inventories/inventory lab -m ping --user bob`

- Enable firewall.
  `ansible-playbook playbooks/firewall.yml --user bob --ask-become-pass`

- Set timezone.
  `ansible-playbook playbooks/timezone.yml --user bob --ask-become-pass`

- Restart.
  `ansible-playbook playbooks/restart.yml --user bob --ask-become-pass`

- Update packages.
  `ansible-playbook playbooks/apt.yml --user bob --ask-become-pass`

- Enable the Droplet Console
  `ansible-playbook playbooks/enable-droplet-console.yml --user bob --ask-become-pass`

### Create Domains and DNS Records
- TODO Create a domain. It also sets an A record if ip is provided.

- Setup DNS records. A, CNAME, TXT, MX, NS records if any exists.

  `ansible-playbook playbooks/dns-records.yml -e "site_name=adamontenegro.com" --user bob --ask-become-pass`

### Setup Caddy Server
- Install Caddy and Setup Domains to serve
  `ansible-playbook playbooks/caddy.yml -e "email=me@example.com" --user bob --ask-become-pass`

- Uninstall Caddy.
    `ansible-playbook playbooks/caddy-uninstall.yml --user bob --ask-become-pass`

- Reload Caddy. Requires `DO_API_TOKEN` to be set.
    `ansible-playbook playbooks/caddy-reload.yml --user bob --ask-become-pass`

### Deliver Static Sites to Server
- Build
  `ansible-playbook playbooks/build.yml -e "site_name=adamontenegro.com" --user bob --ask-become-pass`

- Deliver site `dist` folder to server.
  `ansible-playbook playbooks/deliver.yml -e "site_name=adamontenegro.com site_version=v1.9.0" --user bob --ask-become-pass`

- Deliver site `dist` folder to server. Site name is not same as project folder name.
  `ansible-playbook playbooks/deliver.yml -e "site_name=adamontenegro.com project_name=ada-project-folder site_version=v1.9.0" --user bob --ask-become-pass`

### Deploy Sites
- Deploy a version.
  `ansible-playbook playbooks/deploy.yml -e "site_name=adamontenegro.com site_version=v1.9.0" --user bob --ask-become-pass`

- Read deployment log.
  `ansible-playbook playbooks/deployment-log.yml -e "site_name=adamontenegro.com limit=5" --user bob --ask-become-pass`

## Tips
To target two inventory sources from the command line:
`ansible-playbook get_logs.yml -i inventories/staging -i inventories/production`

To target all inventories from the `inventories` folder:
`ansible-playbook get_logs.yml -i inventories`

To see all the magic variables
`ansible hostname -m ansible.builtin.setup --user bob`
`ansible -m setup hostname --user bob`

See inventory variables for a host
`ansible-inventory --list --yaml`
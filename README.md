# Azure Sensible

This repository provides a set of example, template
[Terraform](https://www.terraform.io/) and [Ansible](https://www.ansible.com)
files for deploying and configuring Azure virtual machines.

## Why might you want to use this

Through using and building upon these examples you will find that your
deployment is

- Fast (no forms or pointing and clicking in your browser required)
- Reproducible (as long as you keep your configuration files you can tear down
  and redeploy your environment on demand)
- Secure (public key authentication by default with optional two-factor
  authentication)
- Hackable (we aim to provide a good starting point for building the environment
  you need)
- [Permissively licensed](./LICENSE) (you are free to copy, use and modify this
  code as well as to merge it with your own)

## How to use this repository

The repository is split into two directories [terraform](./terraform) and
[ansible](./ansible) which contain the Terraform and Ansible files respectively.
Terraform is used to deploy the Azure resources (virtual machines, disks, public
IP address, _etc._) and Ansible is used to configure the virtual machine.

### Get the code

Download and unzip the [latest
release](https://github.com/alan-turing-institute/azure-sensible/releases/latest)
or clone this repository

```
$ git clone https://github.com/alan-turing-institute/azure-sensible.git
```

### Requirements

Before you start, you will need to install some dependencies,

- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- [Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

Additionally for generating QR code images to be scanned with an authenticator
app you will need,

- [Python > 3.6](https://wiki.python.org/moin/BeginnersGuide/Download)
- [qrencode](https://fukuchi.org/works/qrencode/) (which you will likely find in
  you distributions repositories or
  [brew](https://formulae.brew.sh/formula/qrencode))

### Terraform, provisioning your virtual machine

To use terraform to deploy infrastructure on Azure, you will first need to
authenticate using the Azure CLI

```
$ az login
```

which will launch a browser prompting you to login.

Then you will need to enable the subscription you want to deploy the VM into.
Terraform will use your enabled-by-default subscription.

```
$ az account set --subscription <Subscription Name or ID>
```

To see a list of subscriptions available to you, run: `az account list --output table`

Next you can configure your deployment by editing
[`terraform/terraform.tfvars`](terraform/terraform.tfvars). This file has
comments explaining the configuration options and their default values.

Initialise terraform

```
$ cd terraform
$ terraform init
```

Plan your changes

```
$ terraform plan
```

this will print a list of changes to your terminal so you can see what terraform
will do. Run the terraform plan with

```
$ terraform apply
```

> :warning: Warning
>
> The Terraform plan generates an SSH key for the Ansible admin account. The
> private key is [stored
> unencrypted](https://registry.terraform.io/providers/hashicorp/tls/latest/docs/resources/private_key)
> in the Terraform state file. This is not a secure if you intend on [sharing
> the terraform state](https://www.terraform.io/docs/state/remote.html) and
> should be replaced if you intend on doing so.

### Ansible, configuring your virtual machine

Ansible uses an inventory file to declare managed nodes and arrange them into
groups. The terraform plan will have created an inventory for you specifying
your virtual machine and how to connect to it in the `ansible` directory.

Similarly to terraform, there is a variables file with some options regarding
how Ansible will configure your virtual machine. Edit
[`ansible/ansible_vars.yaml`](ansible/ansible_vars.yaml), as before there are
comments to explain the options.

You can use [`scripts/generate_password.py`](scripts/generate_password.py) to
create compatible password hashes for your users without displaying the password
as plain text. See the [README](scripts/README.md#generating-password-hashes)
for instructions.

Install the required ansible modules from [Ansible
Galaxy](https://galaxy.ansible.com)

```
$ cd ../ansible
$ ansible-galaxy install -r requirements.yaml
```

Now run the playbook on the inventory generated by Terraform to configure your
virtual machine

```
$ ansible-playbook -i inventory.yaml playbook.yaml
```

### Optional: generating QR code images

If the option `totp` was `true` in `ansible_vars.yaml` the Ansible play will
have created a file in the ansible directory called `totp_hashes.txt`. This file
contains the information needed to generate QR code images for each user.

To generate the QR code images run the included Python script

```
$ ./scripts/generate_qr_codes.py
```

There will now be a set of PNG files in your current directory, one for each
user, with file names in the format `<username>.png`. These can be distributed
to each user so that they may scan the QR code with their authenticator app.

### Connect to your virtual machine

Both the Terraform plan and the Ansible playbook will finish by printing the
public IPv4 address of your virtual machine. You can connect to the machine
via SSH using this IP address and the credentials of a user your created

```
$ ssh <username>@<ip_address> -i <path_to_private_keyfile>
```

### Destroy the resources

When you are finished, you can destroy the resources using Terraform. From the
terraform directory run

```
$ terraform destroy
```

This will delete all Azure resources and any data stored on these resources will
be lost.

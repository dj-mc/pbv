# pbv

**P**uppet, **B**olt, and **V**agrant

(Bolt is not in action yet)

---

## Puppet

Manage and automate the configuration of servers.

Puppet code is a declarative DSL (via Ruby).  
Declarative languages describe "what" instead of "how".

The Puppet's `primary server` automates the configuration
of its multiple `agents'` states. The primary server stores
the Puppet code that defines the desired state. An agent
retrieves, translates, and executes the code as commands to
its specified systems (model-driven, pull-based workflow).

```log
1. Agent ---> Facts ---> Primary
2. Primary -> Catalog -> Agent
3. Agent ---> Report --> Primary
4. Primary -> Report ✓
```

Puppet is available as `puppetserver`, `puppet-agent`, and `puppetdb`.

---

## Goals

### Consistency

- validate applied config to achieve desired state
- assume state is applied, identify the cause of failure
- update the model to ensure the problem doesn't occur again

### Automation

- goal: maintain servers to achieve desired state
- but what if there's hundreds/thousands of servers?
- ease scaling infrastructure without doing so manually

### Infrastructure as Code

- combine software development and its structural operations
- adopt version control, peer review, automated testing, CI/CD

### Idempotent Workflow

- run continuously, always get the same result
- ensure the infrastructure's state matches the desired state
- guarantee a system's desired state by repeatedly applying code
- if a system's state deviates, Puppet will reapply its code

---

## Using Puppet Bolt (with Vagrant)

- orchestration tool to automate infrastructure maintenance
- deploy apps, troubleshoot servers, patch systems, stop/restart services
- agentless: connect via SSH, no agent installation required
- use commands, scripts, and applied Puppet code (manifest blocks)
- pluggable inventory system (nice to use with Vagrant)

---

Assuming your real machine is running Ubuntu:

1. See installation instructions [here](https://puppet.com/docs/bolt/latest/bolt_installing.html#install-bolt-on-debian)

2. Like Ansible, let's use [Vagrant](https://github.com/dj-mc/ansi#what-vagrant-can-do) to prototype infrastructure with Bolt

3. Ensure [VirtualBox](https://github.com/dj-mc/ansi#using-virtualbox) or another VM provider is installed

Let's setup the project structure:

Let's also decide to use `generic/rocky8` on the VM (CentOS 8 is EOL).
Rocky Linux uses `rpm` instead of `apt`, if you're from Debian.

Initial artifact generation.  
To devs using this repo: no need to regenerate, skip it.

```bash
bolt project init pbv
vagrant init generic/rocky8
```

Start the VM `vagrant up`.

---

## Additional (puppetserver) Configuration

Install `puppetserver`:

```bash
sudo su -

yum update -y
rpm -Uvh https://yum.puppet.com/puppet-release-el-8.noarch.rpm
yum install -y puppetserver git

# Check version
puppetserver --version
```

---

Configure memory usage:

```bash
sudo su -

vim /etc/sysconfig/puppetserver

# Change:
# JAVA_ARGS="-Xms2g -Xmx2g
# to:
# JAVA_ARGS="-Xms512m -Xmx512m

systemctl start puppetserver
systemctl status puppetserver
# Active: active (running)
systemctl enable puppetserver
```

---

Append `vm.hostname` as `agent`:

```bash
vim /etc/puppetlabs/puppet/puppet.conf
```

```conf
[agent]
server = master.puppet.vm
```

---

Append path to puppet binary:

```bash
cd ~
vim .bash_profile
```

```bash
PATH=$PATH:$HOME/bin
PATH=$PATH:/opt/puppetlabs/puppet/bin
```

---

Install `r10k`, a code management tool:  
https://puppet.com/docs/pe/2019.8/r10k.html

```bash
gem install r10k
```

---

Test `puppet agent`:

```bash
puppet agent -t
```

---

Add a control repository:

```bash
mkdir /etc/puppetlabs/r10k
vim /etc/puppetlabs/r10k/r10k.yaml
```

```yaml
---
# Look for modules w/ r10k
:cachedir: '/var/cache/r10k'

:sources:
        :dj-mc:
                # Source control
                remote: 'https://github.com/dj-mc/puppet-control.git'
                # r10k will look here for code and modules
                basedir: '/etc/puppetlabs/code/environments'
```

```bash
# Deploy the env
r10k deploy environment -m

# Repository location
cd /etc/puppetlabs/code/environments/
```

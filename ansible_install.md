###
Method 1 Best Approch

```
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
ansible --version
```
```
ansible localhost -m ping
```

```
ansible all -i inventory -m ping
```

###

Method 2 

```
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
pip install ansible
ansible --version
```

```
export PATH=$PATH:/home/ubuntu/.local/bin
```

```
echo 'export PATH=$PATH:/home/ubuntu/.local/bin' >> ~/.bashrc
```

```
source ~/.bashrc
```

```
ansible --version
```

```
ansible-config init --disabled > ansible.cfg
```

```
ansible --version
```

```
ansible localhost -m ping
```

```
sudo mkdir -p /etc/ansible
```

```
sudo mv /home/ubuntu/ansible.cfg /etc/ansible/ansible.cfg
```

```
ansible --version
```

```
Task:

Install Ansible on the control node or master (Ubuntu system 22.04 LTS).

Make sure ansible.cfg file is available or not if not explore solution for that.

Create an Ansible inventory file with the following structure:

Two servers in the web group

One server in the db group

Configure common SSH access details using inventory variables.

Use Ansible to:

Ping only the web servers

Ping only the db server

Verify that all reachable hosts return a pong response.
```

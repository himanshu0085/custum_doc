```
sudo apt update
```

```
sudo apt install -y python3 python3-pip python3-venv
```

```
pip install ansible
```

```
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
sudo mkdir -p /etc/ansible
sudo mv /home/ubuntu/ansible.cfg /etc/ansible/ansible.cfg
ansible --version

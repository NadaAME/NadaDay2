- Connect to the master node via SSH and run the following commands.

sudo yum update -y
sudo amazon-linux-extras install ansible2 -y

- Confirm Installation
ansible --version

- Configure Ansible on the Master Node
$ sudo su
$ cd /etc/ansible
$ ls
$ vim hosts
[webservers]
node1 ansible_host=<node1_ip> ansible_user=ec2-user
node2 ansible_host=<node2_ip> ansible_user=ec2-user

[all:vars]
ansible_ssh_private_key_file=/home/ec2-user/<pem file>

[defaults]
host_key_checking = False

$ ansible all --list-hosts
$ ansible webservers --list-hosts


- Ansible groups
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com

- Testing hosts
ansible all -m ping
ansible webservers -m ping
ansible node1 -m ping

# Check doc
ansible-doc ping

# single line output
ansible all -m ping -o

# removing warning
sudo vim /etc/ansible/ansible.cfg

- Running commands on hosts
ansible webservers -m shell -a "systemctl status sshd"

ansible webservers -m command -a 'df -h'

ansible webservers -a 'df -h'

vi testfile    # Create a text file name "testfile"
  "This is a test file."

ansible webservers -m copy -a "src=/home/ec2-user/testfile dest=/home/ec2-user/testfile"

# Verify if the file got copied in node1
$ ansible node1 -m shell -a "ls | cat *"

$ ansible node1 -m shell -a "echo Hello DevOps > /home/ec2-user/testfile2 ; cat testfile2"

- Using Shell


$ ansible webservers -b -m shell -a "yum -y update ; yum -y install nginx ; service nginx start; systemctl enable nginx"

# If the above command gives an error complaining about the existance of the package, try the command below.


ansible webservers -b -m shell -a "amazon-linux-extras install -y nginx1 ; systemctl start nginx ; systemctl enable nginx"

# Create one more ubuntu node to test. Uncomment the Node3 in CF and add Node3 in /etc/ansible/hosts
ansible node3 -b -m shell -a "apt update -y ; apt-get install -y nginx ; systemctl start nginx; systemctl enable nginx"

$ ansible webservers -b -m shell -a "yum -y remove nginx"


- Using Yum and Package Module
ansible-doc yum

# Bring back nginx
ansible webservers -b -m yum -a "name=nginx state=present"

# Start back nginx
$ ansible webservers -b -m shell -a "systemctl start nginx ; systemctl enable nginx"

# Remove nginx in node3
ansible node3 -b -m shell -a "apt-get remove nginx -y"

# difference of ```yum``` and ```package``` modules. Package automatically detects underlying OS and uses yum or apt.
$ ansible -b -m package -a "name=nginx state=present" all

- Using Your Own Inventory

$ vim inventory

[webservers]
node1 ansible_host=<node1_ip> ansible_user=ec2-user

[webservers:vars]
ansible_ssh_private_key_file=/home/ec2-user/<YOUR-PEM-FILE-NAME>.pem


- Own inventory with playbook

ansible -i inventory -b -m yum -a "name=httpd state=present" node1
ansible -i inventory -b -m yum -a "name=httpd state=absent" node1

# If absent didn't work
ansible -i inventory -b -m shell -a "netstat -plten | grep 80" node1
ansible -i inventory -b -m shell -a "fuser -k 8080/tcp" node1

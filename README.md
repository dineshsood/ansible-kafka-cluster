# ansible-kafka-cluster
Ansible script to create 3 node kafka cluster on RHEL 7.5 by providing either 3 IP address or dns server names 

1. Create **<script_location>** folder inside the Host machine, where you will run Ansible script
   * pull "**ansible-kafka-cluster**" project into this folder from Github
   * `mkdir <script_location>/SSH` folder     
   * Copy AWS "<aws-private-key>.pem" file into <script_location>/SSH location. Which will be used by Ansible script to login into Servers
   * Open **<script_location>/ansible-kafka-cluster/group_vars/all.yml** file and replace **\<aws-private-key\>** with name of your **_aws private key file name_** in line **/home/{{ ansible_user }}/SSH/\<aws-private-key>\.pem**                         

2. Make sure all the three host machines can talk to each other on port 2181,2888,3888,9092 and 22. Also, host machine where Ansible script is running can talk to all the three host machines on port 22.

3. Add three private Ip addresses in the `<script_location>/ansible-kafka-cluster/hosts` inventory file

[kafka-broker-1]                                        
_**Priavte-IPv4-1**_                                         # Provide AWS EC2 private IP address

[kafka-broker-2]                   
_**Priavte-IPv4-2**_                                         # Provide AWS EC2 private IP address

[kafka-broker-3]                
_**Priavte-IPv4-3**_                                         # Provide AWS EC2 private IP address

### Have a look at all the tasks in the `ansible-kafka-cluster\roles\kafka-cluster-install\tasks` folder
The execution sequence is defined in the main.yml file

Please go through all the tasks in this folder and comment certain tasks which are not required. 
For example:

If swappiness is already set to 1. Then, comment task "- name: set swappiness to 1". 

# Install Ansible on RHEL 7 
Refer to this site 
https://devopsmates.com/ansible-installation-and-configuration-on-redhatcentos-7/

#### I used below commands to install Ansible
`sudo yum update -y`             
`sudo easy_install pip`            
`sudo pip install paramiko PyYAML jinja2 httplib2`                   
`sudo pip install ansible`

# Run Ansibe Kafka install script
`sudo su -`                
`cd <script_location>/ansible-kafka-cluster`                                                      
`export ANSIBLE_HOST_KEY_CHECKING=False`                                     
`ansible-playbook -vvv  -i ./hosts  ./kafka-cluster-playbook.yml`

# Test Kafka cluster is up and running

### SSH into all three host [kafka-broker-1], [kafka-broker-2] and [kafka-broker-3] machines and run below commands

##### Zookeeper Test
`echo "ruok" | nc IpAddrees1 2181 ; echo`                  
Expect imok

`echo "ruok" | nc IpAddrees2 2181 ; echo`         
Expect imok

`echo "ruok" | nc IpAddrees3 2181 ; echo`            
Expect imok

##### Kafka Test
`nc -vz IpAddrees1 9092`                 
Expect "Connection Successful" 

`nc -vz IpAddrees2 9092`

`nc -vz IpAddrees3 9092`

# Create Topics, create consumer and producer
Run these commands from the kafka installation directory.

##### Create topic -> SSH into [kafka-broker-1] host machine and run below command
`bin/kafka-topics.sh --zookeeper zookeeper1Ip:2181,zookeeper2Ip:2181,zookeeper3Ip:2181/kafka --create --topic second_topic --replication-factor 3 --partitions 3`

##### we can publish data to Kafka using the bootstrap server list. SSH into [kafka-broker-1] host machine and run below command. Hist enter, let the producer running
`bin/kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic second_topic`

##### we can read data using any broker too! SSH into [kafka-broker-2] host machine and run below command
`bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 --topic second_topic --from-beginning`

##### we can read data using any broker too! SSH into [kafka-broker-3] host machine and run below command
`bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 --topic second_topic --from-beginning`

Now, go back to [kafka-broker-1] host, where kafka producer is running and type some text and see it coming on [kafka-broker-2] and [kafka-broker-3] host. It, means kafka cluster is running fine.

# ansible-kafka-cluster
Ansible script to create 3 node kafka cluster on RHEL 7.5 by providing either 3 IP address or dns server names 

Add three Ip addresses or dns names in the `/config/hosts` file

[kafka-broker-1]                                        
1.2.3.4                            # Provide AWS private IP address

[kafka-broker-2]                   
2.3.4.5                            # Provide AWS private IP address

[kafka-broker-3]                
3.4.5.6                            # Provide AWS private IP address

### Have a look at all the tasks in the `ansible-kafka-cluster\roles\kafka-cluster-install\tasks` folder
The execution sequence is defined in the main.yml file

Please go through all the tasks in this folder and comment certain tasks which are not required. 
For example:

If swappiness is already set to 1. Then, comment task "- name: set swappiness to 1". 

#Note :                   
#####If you run it multiple times then some of the steps will make changes to the server config again and again.
For example: task like "`- name: Create Zookeeper myid file in each instance`". which runs echo command and
 Ansible has no way to know if its already ran or not. So, it may run it again and you will end up with duplicate
 entries in the target file. Just comment this task, if it ran successfully the first time and rerun the Ansible. 
 I know there is way to mitigate this issue but I could not get time to do that. If you know the way then you are 
 free to update this code.

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
'mkdir <script_location>/SSH' #copy AWS ".pem" file here. Which will be used in the Ansible to login into Servers
'export ANSIBLE_HOST_KEY_CHECKING=False'        
`ansible-playbook -vvv  -i ./hosts  ./kafka-cluster-playbook.yml`

# Test Kafka cluster is up and running


##### Zookeeper Test
`echo "ruok" | nc IpAddrees1 2181 ; echo`           
Expect imok

`echo "ruok" | nc IpAddrees2 2181 ; echo`

`echo "ruok" | nc IpAddrees3 2181 ; echo`

##### Kafka Test
`nc -vz IpAddrees1 9092`                 
Expect "Connection Successful" 

`nc -vz IpAddrees2 9092`

`nc -vz IpAddrees3 9092`


# Create Topics, create consumer and producer
Run these commands from the kafka installation directory.

##### Create topic
`bin/kafka-topics.sh --zookeeper zookeeper1Ip:2181,zookeeper2Ip:2181,zookeeper3Ip:2181/kafka --create --topic second_topic --replication-factor 3 --partitions 3`

##### we can publish data to Kafka using the bootstrap server list!
`bin/kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic second_topic`

##### we can read data using any broker too!
`bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 --topic second_topic --from-beginning`



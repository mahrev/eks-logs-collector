### <span style="font-family: times, serif; font-size:16pt; font-style:italic;"> EKS Logs Collector 

<span style="font-family: calibri, Garamond, 'Comic Sans MS' ;">This project was created to collect Amazon EKS log files and OS logs for troubleshooting Amazon EKS customer support cases.</span>

#### *Usage*
Run this project as the root user:
```
curl -O https://raw.githubusercontent.com/nithu0115/eks-logs-collector/master/eks-log-collector.sh
sudo bash eks-log-collector.sh
```

Confirm if the tarball file was successfully created (it can be .tgz or .tar.gz)

#### *Retrieving the logs*
Download the tarball using your favourite Secure Copy tool.

#### *Example output*
The project can be used in normal or enable_debug(**Caution: enable_debug will prompt to confirm if we can restart Docker daemon which would kill running containers**).

```
# sudo bash eks-log-collector.sh --help
USAGE: eks-log-collector --mode=collect|enable_debug
       eks-log-collector --help

OPTIONS:
     --mode  Sets the desired mode of the script. For more information,
             see the MODES section.
     --help  Show this help message.

MODES:
     collect        Gathers basic operating system, Docker daemon, and Amazon
                    EKS related config files and logs. This is the default mode.
     enable_debug   Enables debug mode for the Docker daemon
```
#### *Example output in normal mode*
The following output shows this project running in normal mode.

```
sudo bash eks-log-collector.sh

	This is version 0.0.4. New versions can be found at https://github.com/awslabs/amazon-eks-ami

Trying to collect common operating system logs... 
Trying to collect kernel logs... 
Trying to collect mount points and volume information... 
Trying to collect SELinux status... 
Trying to collect iptables information... 
Trying to collect installed packages... 
Trying to collect active system services... 
Trying to collect Docker daemon information... 
Trying to collect kubelet information... 
Trying to collect L-IPAMD information... 
Trying to collect sysctls information... 
Trying to collect networking infomation... 
Trying to collect CNI configuration information... 
Trying to collect running Docker containers and gather container data... 
Trying to collect Docker daemon logs... 
Trying to archive gathered information... 

	Done... your bundled logs are located in /opt/log-collector/eks_i-0717c9d54b6cfaa19_2019-02-02_0103-UTC_0.0.4.tar.gz
```


### Collecting EKS Worker Node(s) logs using SSM

#### *Prerequisites*:

1. Configure AWS CLI on the system where you will run the below commands. The IAM entity (User/Role) should have SSM permissions.

2. SSM agent should be installed and running on Worker Node(s).

3. Worker Node(s) should have required permissions to communicate with SSM service. IAM managed role `AmazonEC2RoleforSSM` will have all the required permission for SSM agent to run on EC2 instances. The IAM managed role `AmazonEC2RoleforSSM` has `S3:PutObject` permission to all S3 resources. 

*Note:* For more granular control of the IAM permission check [AWS Systems Manager Permissions Reference ](https://docs.aws.amazon.com/systems-manager/latest/userguide/auth-and-access-control-permissions-reference.html) link

4. A S3 bucket location is required which is taken as an input parameter to `aws ssm send-command` command, to which the logs should be pushed.


#### *To run `aws ssm send-command` to collect logs from Worker Node(s):*

1. Create the SSM document named "EKSLogCollector" using the following command:

```aws ssm create-document --name "EKSLogCollector" --document-type "Command" --content https://github.com/mahrev/eks-logs-collector/blob/master/eks-ssm-content.json```

2. To execute the bash script in the SSM document and to collect the logs from worker, run the following command: 

```aws ssm send-command --instance-ids <worker node instance ID> --document-name "EKSLogCollector" --parameters "bucketName=<S3 bucket name to push the logs>" --output text```

3. Once the above command is executed successfully, the logs should be present in the S3 bucket specified in the previous step. 



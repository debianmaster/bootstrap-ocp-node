
## Bootstrap OCP node.  (Do not use)



```sh
aws configure
aws ec2 create-security-group --group-name ocp-common --description "OCP Common"
aws ec2 authorize-security-group-ingress --group-name ocp-common --protocol tcp --port 0-65000 --cidr 0.0.0.0/24

aws ec2 run-instances --image-id ami-a3fa16c3 --count 2 --instance-type t2.medium --key-name ck_workshop  --security-groups ocp-common

aws ec2 describe-instances --filters "Name=key-name,Values=ck_workshop" | grep PublicIp | grep PublicIpAddress | awk '{print $2}' | sed 's/"//g' | sed 's/,//g'

52.41.45.165
52.33.111.141
54.70.161.129

ansible all -i hosts -m shell -su --su-user=ec2-user -a "subscription-manager register --username=username --password=password --force" 

ansible all -i hosts -m shell -su --su-user=ec2-user -a "subscription-manager attach --pool=$(subscription-manager list --available --matches "Red Hat OpenShift Container Platform" --pool-only)"

ansible all -i hosts -m shell -su --su-user=ec2-user -a "subscription-manager repos --disable='*'"

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'subscription-manager repos --enable="rhel-7-server-rpms" --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.3-rpms"'

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion'

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'yum -y install atomic-openshift-utils' 
```

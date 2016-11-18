
## Bootstrap OCP node.  (Do not use)


### Setup AWS Instances (Optional)
```sh
AWS_ACCESS_KEY_ID=your_key_id
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
ANSIBLE_HOST_KEY_CHECKING=false

aws configure
aws ec2 create-security-group --group-name ocp-common --description "OCP Common"
aws ec2 authorize-security-group-ingress --group-name ocp-common --protocol tcp --port 0-65000 --cidr 0.0.0.0/24

aws ec2 run-instances --image-id ami-a3fa16c3 --count 2 --instance-type t2.medium --key-name ck_workshop  --security-groups ocp-common

######## ami-eae6458a

echo "[nodes]" > hosts

aws ec2 describe-instances --filters "Name=key-name,Values=ck_workshop" | grep PublicIp | grep PublicIpAddress | awk '{print $2}' | sed 's/"//g' | sed 's/,//g'  >> hosts
```

### Login to master and generate id_rsa.pub
```sh
export OCP_MASTER=$(cat hosts  | head -2 | tail -1)
# Copy public file to id_rsa.pub on same folder
ssh-add <your_aws_pem> 
ssh $OCP_MASTER
sudo su
ssh-keygen
#copy ssh ec2-user@$OCP_MASTER cat /root/.ssh/id_rsa.pub >  id_rsa.pub
ansible-playbook -i hosts copy-ssh-key.playbook  -su --su-user=ec2-user
```

### Bootstrap Node
```sh
ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a "subscription-manager register --username=username --password=password --force" 

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'subscription-manager attach --pool=$(subscription-manager list --available --matches "Red Hat OpenShift Container Platform" --pool-only)'

ansible all -i hosts -m shell -su --su-user=ec2-user -a "subscription-manager repos --disable='*'"

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'subscription-manager repos --enable="rhel-7-server-rpms" --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.3-rpms"'

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion'

ansible all -i hosts -m shell -su --su-user=ec2-user -a 'yum -y install atomic-openshift-utils' 
```

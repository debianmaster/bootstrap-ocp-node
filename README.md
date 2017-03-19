### AWS

```sh
export AWS_ACCESS_KEY_ID=your_key_id
export AWS_SECRET_ACCESS_KEY=your_aws_secret_key
export ANSIBLE_HOST_KEY_CHECKING=false

aws configure
aws ec2 create-security-group --group-name ocp-common --description "OCP Common"
aws ec2 authorize-security-group-ingress --group-name ocp-common --protocol tcp --port 0-65000 --cidr 0.0.0.0/24

aws ec2 run-instances --image-id ami-6f68cf0f --count 4 --instance-type t2.medium --key-name ck_workshop  --security-groups ocp-common

######## ami-eae6458a

echo "[nodes]" > hosts

aws ec2 describe-instances --filters "Name=key-name,Values=ck_workshop" | grep PublicIp | grep PublicIpAddress | awk '{print $2}' | sed 's/"//g' | sed 's/,//g'  >> hosts



scp -r ck_workshop.pem ec2-user@$OCP_MASTER:/tmp




ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a "subscription-manager register --username=my_rh_uname --password=mypass --force" 

ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a 'subscription-manager attach --pool=$(subscription-manager list --available --matches "Red Hat OpenShift Container Platform" --pool-only | tail -1)'

ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a "subscription-manager repos --disable='*'"

ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a 'subscription-manager repos --enable="rhel-7-server-rpms" --enable="rhel-7-server-extras-rpms" --enable="rhel-7-server-ose-3.4-rpms"'

ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a 'yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion'

ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a 'yum -y install atomic-openshift-utils' 


ansible all -i hosts -m shell -su --su-user=ec2-user -u ec2-user -a "subscription-manager unregister  --force" 

ssh ec2-user@$OCP_MASTER
ssh-agent $SHELL
ssh-add /tmp/ck_workshop.pem 
```



## GCP

```sh
gcloud compute instances create "master1" --zone "asia-east1-a" --machine-type "custom-4-16384" --subnet "default" --image "centos-7-v20170223" --image-project "centos-cloud" --boot-disk-size "20" --boot-disk-type "pd-ssd" --boot-disk-device-name "master1"

gcloud compute  firewall-rules create "default-allow-http" --allow tcp:80 --network "default" --source-ranges "0.0.0.0/0" --target-tags "http-server"

gcloud compute firewall-rules create "default-allow-https" --allow tcp:443 --network "default" --source-ranges "0.0.0.0/0" --target-tags "https-server"



gcloud compute instances create "master2" --zone "asia-east1-a" --machine-type "custom-4-16384" --subnet "default" --image "centos-7-v20170223" --image-project "centos-cloud" --boot-disk-size "20" --boot-disk-type "pd-ssd" --boot-disk-device-name "master2"


gcloud compute instances create "master3" --zone "asia-east1-a" --machine-type "custom-4-16384" --subnet "default" --image "centos-7-v20170223" --image-project "centos-cloud" --boot-disk-size "20" --boot-disk-type "pd-ssd" --boot-disk-device-name "master3"

gcloud compute instances create "node1" --zone "asia-east1-a" --machine-type "custom-4-16384" --subnet "default" --image "centos-7-v20170223" --image-project "centos-cloud" --boot-disk-size "60" --boot-disk-type "pd-ssd" --boot-disk-device-name "node1"

gcloud compute instances create "node2" --zone "asia-east1-a" --machine-type "custom-4-16384" --subnet "default" --image "centos-7-v20170223" --image-project "centos-cloud" --boot-disk-size "60" --boot-disk-type "pd-ssd" --boot-disk-device-name "node2"
```

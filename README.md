# ok-shell-tips

## Just some stuff that I'm too bored to write again and too lasy to develop into separate tool

DevOps Automation routines mostly. Maybe some of them lack explanation, but I'm lasy :)

_Warning! Use it strictly on your own risk_ 
_Note! Feel free to add some variation or idea for another approach_

### AWS

### Update user password (your cap)
```shell
aws --profile PROFILE iam update-login-profile --user-name USER --password
```

### Search for something in some ~~trashcan~~ S3 bucket. Second is iterate over buckets with threaded tool
```shell
aws --profile PROFILE s3 ls s3://BUCKET/ --recursive | grep PATTERN
for b in $(s4cmd -r ls | grep ftp | awk '{print $4}') ; do s4cmd -r ls ${b}1465/ ; done
```

### Find subnets that are short of IPs
```shell
aws --profile PROFILE ec2 describe-subnets --filters "Name=vpc-id,Values=VPC_ID" | jq '.Subnets[] | .SubnetId + "=" + "\(.AvailableIpAddressCount)"'
```

### Get all user keys. For blameless security investigation, you know
```shell
for u in $(aws --profile PROFILE iam list-users  | jq ".Users[].UserName" --raw-output); do   aws --profile PROFILE iam list-access-keys --user $u | jq '.AccessKeyMetadata[] | .UserName + ":" + .AccessKeyId' ; done
```

### K8S

### See Events sorted by timestamp
````shell
k get events --sort-by='.lastTimestamp'
````

### Merge kubeconfigs
```shell
cp ~/.kube/config ~/.kube/config.bak && KUBECONFIG=~/.kube/config:./ok-cluster/ok-cluster-eks-a-cluster.kubeconfig kubectl config view --flatten > /tmp/config && mv /tmp/config ~/.kube/config
```

### Add profile to kubeconfig, all of profiles all clusters
```shell
aws eks --profile ok-dev update-kubeconfig --name eks-terra --alias ok-eks
for p in $(aws configure list-profiles) ; do for c in $(aws eks --profile $p list-clusters | jq '.clusters[]' | tr -d '\"') ; do echo $p $c ; aws eks --profile $p update-kubeconfig --name $c --alias $p:$c ; done ; done
```

### Saving ~~private~~ persistent volume
```shell
kubectl patch pv $PV_NAME_i -p \
  '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

### Label all nodes with their type and lifecycle, mark your cattle ;)
```shell
for n in $(kubectl get nodes -o 'jsonpath={.items[*].metadata.name}') ; do lb="" ;  for a in $(kubectl label --list nodes $n | sort | grep -e NodeType -e lifecycle | cut -d= -f 2) ; do lb="${lb}$a" ; done ; kubectl label nodes $n node-role.kubernetes.io/$lb= ; done
```

### Download and build cdebug on a node
```shell
yum install wget -y ; yum install unzip -y ; wget https://github.com/iximiuz/cdebug/archive/refs/heads/main.zip && unzip main.zip && cd cdebug-main/ && yum install go make -y ; make
```

### Gatekeeper stuff 
```shell
for c in $(kubectl api-resources | grep constraints.gatekeeper.sh/v1beta1 | awk '{print $1}') ; do k describe $c ; done | less
````

## MySQL

### Kill somebody's sessions in RDS
```SQL
SELECT CONCAT('CALL mysql.rds_kill(',id,');')
FROM information_schema.processlist
WHERE user='UGLY_BASTARD';
```

### Self-explanatory
```SQL
SHOW OPEN TABLES WHERE In_use > 0;
SHOW ENGINE INNODB STATUS;
```

## Other

### Delete ALL topics in Kafka
```shell
export KFK=KFK_HOST
for t in $(./bin/kafka-topics.sh --bootstrap-server  $KFK:9092 --list) ; do ./bin/kafka-topics.sh --bootstrap-server $KFK:9092 --topic $t --delete ; done
#-||- --describe | grep 'ReplicationFactor:1' ; done
```

### Inframamp
```shell
terraform state pull | inframap generate --connections=false | dot -Tpng > ~/Downloads/schema.png
```
### KUBE-OPS-VIEW
````shell
(kubectl proxy --accept-hosts '.*' &) ; docker run -it -p 8080:8080 -e CLUSTERS=http://docker.for.mac.localhost:8001 hjacobs/kube-ops-view
````

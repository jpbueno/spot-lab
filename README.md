# [LAB] JP's EC2 Spot Lab

## Verify latest Amazon Linux 2 AMI

```bash
aws ec2 describe-images --owners amazon --filters 'Name=name,Values=amzn2-ami-hvm-2.0.????????-x86_64-gp2' 'Name=state,Values=available' --output json | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
```

## Replace latest AMI-ID to to ec2-launch-template.json and execute the following command to create launch template

```bash
aws ec2 create-launch-template --cli-input-json file://launch-template.json
```

## Create ASG with Lowest Price as spot allocation strategy

```bash
aws autoscaling create-auto-scaling-group --cli-input-json file://asg-lowest-price-n-1.json
```

## Create ASG with Capacity Optimized as spot allocation strategy

```bash
aws autoscaling create-auto-scaling-group --cli-input-json file://asg-capacity-optimized.json
```

## Run the following script to query lowest price asg group

```bash
while read i a p m; do s=$(aws ec2 describe-spot-price-history --instance-types $i --availability-zone $a --product-descriptions "Linux/UNIX (Amazon VPC)" --start-time $(date +%FT%TZ) --end-time $(date +%FT%TZ) --output text); echo -e $s'\t'$m | awk -F'\t' '{print $2, $3, $7, $5, ($7-$5)/$7*100}'; done < <(for sir in $(aws ec2 describe-instances --filters "Name=tag:aws:autoscaling:groupName,Values=lowest-price-n-1" "Name=instance-state-name,Values=running" "Name=instance-lifecycle,Values=spot" --query 'Reservations[*].Instances[*].[SpotInstanceRequestId]' --output text); do aws ec2 describe-spot-instance-requests --spot-instance-request-ids $sir --query 'SpotInstanceRequests[*].[LaunchSpecification.InstanceType, LaunchedAvailabilityZone, ProductDescription, SpotPrice]' --output text; done)
```

## Run the following script to query capacity optimized asg group

```bash
while read i a p m; do s=$(aws ec2 describe-spot-price-history --instance-types $i --availability-zone $a --product-descriptions "Linux/UNIX (Amazon VPC)" --start-time $(date +%FT%TZ) --end-time $(date +%FT%TZ) --output text); echo -e $s'\t'$m | awk -F'\t' '{print $2, $3, $7, $5, ($7-$5)/$7*100}'; done < <(for sir in $(aws ec2 describe-instances --filters "Name=tag:aws:autoscaling:groupName,Values=capacity-optimized" "Name=instance-state-name,Values=running" "Name=instance-lifecycle,Values=spot" --query 'Reservations[*].Instances[*].[SpotInstanceRequestId]' --output text); do aws ec2 describe-spot-instance-requests --spot-instance-request-ids $sir --query 'SpotInstanceRequests[*].[LaunchSpecification.InstanceType, LaunchedAvailabilityZone, ProductDescription, SpotPrice]' --output text; done)
```

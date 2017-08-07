### Once instance(s) are created, automagically ssh to them:


```bash
#!/usr/bin/env bash

set -e

MYIPADDRESS=$(curl -s checkip.amazonaws.com)
SG=$(terraform state show aws_security_group.validator | grep ^id | awk '{print $3;}')
KEY=$(terraform state show tls_private_key.default | awk '/BEGIN RSA PRIVATE KEY/{print "-----BEGIN RSA PRIVATE KEY-----";f=1;next} /END RSA PRIVATE KEY/{print "-----END RSA PRIVATE KEY-----";f=0} f')
HOST=$(terraform state show aws_instance.validator | grep ^public_ip | awk '{print $3;}')

aws ec2 authorize-security-group-ingress \
  --protocol tcp --port 22 \
  --group-id $SG \
  --cidr $MYIPADDRESS/32

KEYFILE=$(mktemp -p ~/.ssh/ tmp.XXXXXXXXXX)
echo -n $KEY >> $KEYFILE
chmod 400 $KEYFILE
ssh -i $KEYFILE ubuntu@$HOST
```

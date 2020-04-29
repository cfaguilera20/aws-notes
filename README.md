# SSH Instructions

```bash
chmod 400 <filename>.pem
```

Access to the EC2 instance
```bash
ssh -i <filename>.pem ec2-user@ip
```

Root access
```bash
sudo su
``` 

Update depedencies
```bash
yum update -y
```

Install apache
```bash
yum install httpd -y
service httpd start
```

Restart the service automatically
```bash
chkconfig on
```

Add default page
```bash
cd /var/www/html
nano index.html
```

CLI configure

```bash
aws configure
```

Use `us-east-1` as default region

Calling a command
```bash
aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
```

List - s3 buckets
```bash
aws s3 ls
```

Create - s3 bucket
```bash
aws s3 mb s3://<bucketname>
```

Access aws folder
```bash
cd ~ && cd .aws
```

Look credentials
```bash
cat credentials # It could be a security issue
```

Delete credentials
```bash
rm -rf .aws
```

We can assign a role to this EC2 instance by Attach/Replace IAM Role option, so we can execute aws commands.


Run bash script on load EC2
```bash
#!/bin/bash
yum update -y
yum install httpd -y
service httpd start
chkconfig httpd on
cd /var/www/html
echo "<html><p>Hello from EC2 bash.</p></html>" > index.html
aws s3 mb s3://YOURBUCKETNAMEHERE
aws s3 cp index.html s3://YOURBUCKETNAMEHERE
```

EC2 Meta data
```bash
sudo su
curl https://169.254.169.254/latest/user-data
curl https://169.254.169.254/latest/user-data > bootstrap.txt
cat bootstrap.txt
curl https://169.254.169.254/latest/meta-data
curl https://169.254.169.254/latest/meta-data/public-ipv4
```

Two EC2 instances - EFS
```bash
#!/bin/bash
yum update -y
yum install httpd -y
service httpd start
chkconfig httpd on
yum install amazon-efs-utils -y
```

Add NFS to default EC2's Secury Group 

Instance 1 & 2
```bash
ssh -i <filename>.pem ec2-user@ip
sudo su
cd /var/www/html
mount -t efs -o tls fs-50ee02d3:/ /var/www/html
touch index.html
```

EC2 - RDS - WordPress

```bash
#!/bin/bash
yum install httpd php php-mysql -y
cd /var/www/html
wget https://wordpress.org/wordpress-5.1.1.tar.gz
tar -xzf wordpress-5.1.1.tar.gz
cp -r wordpress/* /var/www/html/
rm -rf wordpress
rm -rf wordpress-5.1.1.tar.gz
chmod -R 755 wp-content
chown -R apache:apache wp-content
service httpd start
chkconfig httpd on
```

WP PWD: admin  WP_PASSWORD

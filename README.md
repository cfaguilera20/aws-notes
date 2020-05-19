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

WP HA - EC2

 ```bash
 #!/bin/bash
yum update -y
yum install httpd -y
amazon-linux-extras install php7.2 -y
cd /var/www/html
echo "healthy" > healthy.html
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/
rm -rf wordpress
rm -rf latest.tar.gz
chmod -R 755 wp-content
chown -R apache:apache wp-content
wget https://s3.amazonaws.com/bucketforwordpresslab-donotdelete/htaccess.txt
mv htaccess.txt .htaccess
chkconfig httpd on
service httpd start
 ```

WP HA - CloudFront

```bash
sudo su
cd /var/www/html/
vi .htaccess  # Update our CloudFront url
```

WP HA - S3

```bash
aws s3 ls
aws s3 cp --recursive /var/www/html/wp-content/uploads s3://cfa-wp-media
aws s3 cp --recursive /var/www/html s3://cfa-wp-code
aws s3 ls s3://<bucketname>
```

WP HA - S3 Sync

```bash
aws s3 sync /var/www/html s3://cfa-wp-code
```

WP HA - Apache rewrite

```bash
cd /etc/httpd/conf
cp httpd.conf httpd.copy.conf
vi httpd.conf # Find for AllowOverride controls what directives may be placed in .htaccess files. Change the value to All
service httpd restart
```

WP HA - S3 Policy

First allow plublic access to the media bucket, then add the following policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
        ],
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME/*"
        ]
    }
  ]
}
```

Create an Application Load Balancer
Create a A record with Alias to ALBWP
Add my EC2 instance as Registered target

Add crontab to sync data
```bash
cd /etc
nano crontab
```

Add the following cron
```bash
*/1 * * * * root aws s3 sync --delete s3://cfa-wp-code /var/www/html 
```

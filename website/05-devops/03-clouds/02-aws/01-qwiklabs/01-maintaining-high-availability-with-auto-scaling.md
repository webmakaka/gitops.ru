---
layout: page
title: Maintaining High Availability with Auto Scaling (for Linux)
description: Maintaining High Availability with Auto Scaling (for Linux)
keywords: Maintaining High Availability with Auto Scaling (for Linux)
permalink: /devops/clouds/aws/qwiklabs/maintaining-high-availability-with-auto-scaling/
---

# Maintaining High Availability with Auto Scaling (for Linux)

https://www.qwiklabs.com/focuses/303?parent=catalog

<h3>Retrieve your host’s public DNS address</h3>
<ol start="3">
<li>On the <span style="background-color:#232f3e;font-weight:bold;font-size:90%;color:white;padding-top:3px;padding-bottom:3px;padding-left:10px;padding-right:10px;">Services</span> menu, click <strong>EC2</strong>.</li>
<li><p>In the navigation pane, click <strong>Instances</strong>.</p></li>
<li><p>In your list of running Amazon EC2 instances, select the instance to display the instance details.</p></li>
<li><p>Copy the <strong>Public DNS (IPv4)</strong> value to your Clipboard. It will look something like <em>ec2-54-84-236-205.compute-1.amazonaws.com</em>.</p></li>
</ol>

<br/>

    // $ chmod 600 ~/<Downloads/qwiklab-l33-5018.pem>
    $ chmod 600 /mnt/dsk1/qwikLABS-L251-6901459.pem

    // $ ssh -i ~/<Downloads/qwiklab-l33-5018.pem> ec2-user@<public DNS>
    $ ssh -i /mnt/dsk1/qwikLABS-L251-6901459.pem  ec2-user@ec2-34-219-67-4.us-west-2.compute.amazonaws.com

<br/>

    $ cat ~/lab-details.txt
    ElasticLoadBalancer, qls-69014-ElasticL-1LILUT7RDXQ60
    AMIId, ami-f0091d91
    KeyName, qwikLABS-L251-6901459
    AvailabilityZone, us-west-2b
    SecurityGroup, qls-6901459-efb8cfd841dcc951-Ec2SecurityGroup-1O5VSN9LPBJ9X

<br/>

### Configure AWS CLI

To find your Access Key and Secret Access Key:

1. Log in to your AWS Management Console.
2. Click on your user name at the top right of the page.
3. Click on the Security Credentials link from the drop-down menu.
4. Find the Access Credentials section, and copy the latest Access Key ID.
5. Click on the Show link in the same row, and copy the Secret Access Key.

<br/>

    $ aws configure
    AWS Access Key ID [None]: [Enter]
    AWS Secret Access Key [None]: [Enter]
    Default region name [None]: us-west-2
    Default output format [None]: [Enter]

<br/>

    $ cat ~/.aws/credentials
    [default]
    aws_access_key_id =

<br/>

    // $ aws autoscaling create-launch-configuration --image-id <PasteYourAMIIdHere> --instance-type t2.micro --key-name <PasteYourKeyNameHere> --security-groups <PasteYourSecurityGroupHere> --user-data file:///home/ec2-user/as-bootstrap.sh --launch-configuration-name lab-lc


    $ aws autoscaling create-launch-configuration --image-id ami-f0091d91 --instance-type t2.micro --key-name qwikLABS-L251-6901459 --security-groups qls-6901459-efb8cfd841dcc951-Ec2SecurityGroup-1O5VSN9LPBJ9X --user-data file:///home/ec2-user/as-bootstrap.sh --launch-configuration-name lab-lc

    Partial credentials found in shared-credentials-file, missing: aws_secret_access_key

<br/>

<p>The parameters for this command are defined as follows:</p>

<ul>
    <li><p><em>image-id:</em> A 64-bit Amazon Linux AMI.</p></li>
    <li><p><em>instance-type:</em> An EC2 Instance type. The t2.micro instance type is used here.</p></li>
    <li><p><em>key-name:</em> The name of an EC2 Key Pair created by *qwik*LAB™ for you.</p></li>
    <li><p><em>security-groups:</em> The EC2 security group(s) created for you in the lab via CloudFormation.</p></li>
    <li><p><em>launch-configuration-name:</em> The name of this Auto Scaling Launch Configuration. <em>lab-lc</em> is used here.</p></li>
</ul>

<br/>

    $ cat /home/ec2-user/as-bootstrap.sh

```
#!/bin/sh
yum -y install httpd php mysql php-mysql
chkconfig httpd on
/etc/init.d/httpd start
cd /tmp
wget http://us-east-1-aws-training.s3.amazonaws.com/self-paced-lab-4/examplefiles-as.zip
unzip examplefiles-as.zip
mv examplefiles-as/* /var/www/html
```

<br/>

### Create an Auto Scaling Group

In this section, you will create a new Auto Scaling group in your current region and Availability Zone. The group will ensure that there is always one server running by establishing a minimum Auto Scaling group size of one. You will also specify that the maximum number of servers in this group must not exceed five.

    // $ aws autoscaling create-auto-scaling-group --auto-scaling-group-name lab-as-group --availability-zones <PasteYourAvailabilityZoneHere> --launch-configuration-name lab-lc --load-balancer-names <PasteYourElasticLoadBalancerHere> --max-size 4 --min-size 1

<br/>

    $ aws autoscaling create-auto-scaling-group --auto-scaling-group-name lab-as-group --availability-zones us-west-2b --launch-configuration-name lab-lc --load-balancer-names qls-69014-ElasticL-1LILUT7RDXQ60 --max-size 4 --min-size 1


    Partial credentials found in shared-credentials-file, missing: aws_secret_access_key

<p><strong>Important</strong> Replace the &lt;PasteYourAvailabilityZoneHere&gt; placeholder from the Connection Details of the Qwiklabs page and also be sure to use the load balancer value returned by the cat ~/lab-details.txt command. This parameter is limited to 35 characters. If you use the fully qualified DNS name of the load balancer, your Auto Scaling group will not function properly.</p>

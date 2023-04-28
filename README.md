# NAT-server

![image](https://user-images.githubusercontent.com/66196388/206446762-b3cf7b31-ab21-4163-9987-92a99111bfdb.png)


## Configuring NAT-server on AWS

```bash
- create vpc vijay-vpc(10.0.0.0/16)
```

```bash
- create public-subnet(10.0.1.0/24)(AZ-1)
         private-subnet(10.0.2.0/24)(AZ-2)
```

```bash
- when you create a new vpc, a new main RT will be created by default and has a route which routes 10.0.0.0/16 to local, means which doesnt have internet access, so it is always recommended to create new custom route tables and leave main RT as it is
```

```bash
- create two custom route tables(vijay-public-RT and vijay-private-RT)
- vijay-public-RT add to vpc(vijay-vpc)
- vijay-private-RT add to vpc(vijay-vpc)
```

```bash
- create internet Gateway(IGW)(vijay-IGW)
- there will be default internet gateway, You can attach only one internet gateway to a VPC at a time
- attach vijay-IGW to vijay-vpc
```

```bash
- create one private instance(vijay-private-instance)
- select vijay-vpc
- select private-subnet(10.0.2.0/24)(AZ-2)
- disable auto-assign public ipv4 address
- open 22 port
- anyway when you create a subnet, by default it has no internet access because auto-assign public ip will be disabled by default, but for default subnet auto-assign public ip will be enabled so we can access internet from default subnet
- this private instance will not have internet access and we cant even ssh to it
```

```bash
- create NAT instance(NAT server)
- NAT is like a router which allows internet traffic to private devices in the network, it translates private ip to public ip in internet and same public ip to private ip in the private network
- NAT only allows outbound traffic to internet and it blocks inbound traffic from the internet 
- NAT instance/NAT gateway always should reside in the public subnet and allow internet traffic for instances in private subnet
- make use of amazon community ami's - AMI naming = "amzn-ami-vpc-nat"
- select amzn-ami-vpc-nat community ami's
- example:- "amzn-ami-vpc-nat-2018.03.0.20210721.0-x86_64-ebs" in us-west-2
- ami-07516a08e3c79b372 
- select vijay-vpc
- public-subnet(10.0.1.0/24)(AZ-1)
- auto-assign public-ip enable 
- and we can also attch elastic ip to this instance
- enable port 22 
  SSH                    TCP         22               0.0.0.0/0
 - enable ALL ICMP-IPV4 
  ALL ICMP-IPV4          ICMP        0-65535          0.0.0.0/0
  
  - you can allow 80 and 443 to update private servers which are behind NAT instance
  
- this public instance will also not have internet access and we cant even ssh to it
```

```bash
- goto Route tables
- add route in public-RT 
- edit routes
  0.0.0.0./0 OR 10.0.2.0/24 - vijay-IGW 
- edit subnet associations
- and associate public subnet to public-RT
```

```bash
- add route in private-RT 
- edit routes
  0.0.0.0./0 OR 10.0.2.0/24 - instance(NAT-server instance id)
- edit subnet associations
- and associate private subnet to private-RT
```

```bash
- select NAT-server and goto actions > "networking" > change source/destination check > click on "stop" > save
- now connect to NAT server 
- ssh -i "key.pem" ec2-user@34.221.25.136
```

```bash
- vijay-private-instance has no public ip associated and also it is in private subnet
- but both instances(vijay-private-instance and NAT server) are within vpc and main vpc RT CIDR points to local, so locally they can communicate within VPC
- ssh into NAT-server
[ec2-user@ip-10-0-1-60 ~]$ ping google.com
PING google.com (142.251.33.78) 56(84) bytes of data.
64 bytes from sea09s28-in-f14.1e100.net (142.251.33.78): icmp_seq=1 ttl=44 time=6.04 ms
64 bytes from sea09s28-in-f14.1e100.net (142.251.33.78): icmp_seq=2 ttl=44 time=6.02 ms
```

```bash
- so from NAT server you can access internet
- vi /home/ec2-user/key.pem
- copy and paste the pem key which is used while creation of instances
- chmod 400 key.pem
- ssh -i key.pem ec2-user@10.0.2.70
- now you are inside private instance(vijay-private-instance)
[ec2-user@ip-10-0-2-70 ~]$ ping google.com
PING google.com (142.251.33.110) 56(84) bytes of data.
64 bytes from sea30s10-in-f14.1e100.net (142.251.33.110): icmp_seq=1 ttl=103 time=8.79 ms
64 bytes from sea30s10-in-f14.1e100.net (142.251.33.110): icmp_seq=2 ttl=103 time=8.80 ms

so by using NAT-server on public subnet we can acecss internet for isntances inside private subnet
dot
```

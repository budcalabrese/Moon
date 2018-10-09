# Working with VPC Endpoint Services (AWS PrivateLink)

Recently I started a new role moving on-prem infrastructure to aws. One of my current tasks is to move Puppet from on-prem to aws. The catch is Puppet has to live in a different aws account thna where the ec2 instances will be built. 

Peering was a option but not ideal. After more google searching I found AWS PrivateLink which seemed like it could be a solid option. https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html  

After reading thrnought the documentation I started to get a better understand of data flow. 

![basic_tuckernet](https://github.com/budcalabrese/terraform/blob/master/img/basic_privatelink.jpeg)

### From here you can see you it is broken into two parts. 
1.  Service provider  
2.  Service consumer  


#### Service provider steps:
- Setup a nlb (network load balancer)
- Endpoint service pointing to the nlb

#### Service consumer steps:
- Create endpoint that is pointed to the service name of the service endpoint  
  - Created on the endpoint service in acount #1 in this example. 

### Some Gotcha's to watch out for:
- Each endpoint is for a single service
- Instead of using the AWS provided DNS use a CNAME to make communication easier
- Must whitelist arn of consuming account (example account#2 in diagram)
  - Whitelisting can be done by whole account or by user
- On the service provider (endpoint service) you have to accept the consumer account request
- Enable cross-zone communication on the load balancer  


Now that we have the basics covered lets look at automating this creation with Terraform 

### Creating VPC Endpoint Service

Really the only new part is the module `aws_vpc_endpoint_service`. Everything else is normal loadbalancer modules in Terraform. The break down of the terraform is:
- Create loadbalancer
- Create a loadbalancer target group
- Attacing hosts to loadbalancer target group
- Create loadbalancer listener
- Create vpc enpoint service

```
resource "aws_lb" "puppet_master" {
  name               = "puppet-master-lb"
  internal           = true
  load_balancer_type = "network"
  subnets            = ["${data.aws_subnet_ids.private.ids}"]
  enable_cross_zone_load_balancing = true

  tags {
    Environment = "alpha"
  }
}

resource "aws_lb_target_group" "puppet_master" {
  name     = "puppet-master"
  port     = "${var.port}"
  protocol = "${var.protocol}"
  vpc_id   = "${var.vpc_id}"
}

resource "aws_lb_target_group_attachment" "puppet_master" {
  target_group_arn = "${aws_lb_target_group.puppet_master.arn}"
  target_id = "${var.host}"
  port = "${var.port}"
}

resource "aws_lb_listener" "puppet_master" {
  load_balancer_arn = "${aws_lb.puppet_master.arn}"
  port              = "${var.port}"
  protocol          = "${var.protocol}"

  default_action {
    type             = "forward"
    target_group_arn = "${aws_lb_target_group.puppet_master.arn}"
  }
}

resource "aws_vpc_endpoint_service" "puppet_master" {
  acceptance_required = true
  network_load_balancer_arns = ["${aws_lb.puppet_master.arn}"]
  allowed_principals = ["${var.allowed_arn}"]
}
```  

### Creating VPC Endpoint 

The Terraform is pretty straight foward here. 
- Create vpc endpoint
- Create a route53 cname record for the dns entry 

*For endpoint communication think of a vpn. You have to use the dns name of the endpoint. To make a friendly dns name we create a cname record*

```
resource "aws_vpc_endpoint" "puppet_master" {
  vpc_id            = "${var.vpc_id}"
  service_name      = "${var.puppet_master_service}"
  vpc_endpoint_type = "Interface"

  security_group_ids = [
    "${var.puppet_sg}"
  ]
  
  subnet_ids          = ["${element(data.aws_subnet_ids.private.ids, count.index)}"]
}

data "aws_route53_zone" "internal" {
  name         = "${var.dns_zone}"
  private_zone = true
  vpc_id       = "${var.vpc_id}"
}

resource "aws_route53_record" "puppet_master_service" {
  zone_id = "${data.aws_route53_zone.internal.zone_id}"
  name    = "puppetmaster.${data.aws_route53_zone.internal.name}"
  type    = "CNAME"
  ttl     = "300"
  records = ["${lookup(aws_vpc_endpoint.puppet_master.dns_entry[0], "dns_name")}"]
}
```

# AUTOMATE-INFRASTRUCTURE-WITH-IAC-USING-TERRAFORM-PART-2
This is a continuation from the previous project 16. Here , we will need to set up subnets just like what we did in earlier manually.

## Networking - Private subnets & best practices
- Create 4 private subnets keeping in mind following principles:
  - Make sure you use variables or length() function to determine the number of AZs.
  - Use variables and cidrsubnet() function to allocate vpc_cidr for subnets.
  - Keep variables and resources in separate files for better code structure and readability.
  - Tag all the resources you have created so far. Explore how to use format() and count functions to automatically tag subnets with its respective number.
    
  **Note**: You can add multiple tags as a default set. for example, in out terraform.tfvars file we can have default tags defined.

        tags = {
          Environment      = "production" 
          Owner-Email     = "infradev-segun@darey.io"
          Managed-By      = "Terraform"
          Billing-Account = "1234567890"
        }

- Now you can tag all your resources using the format below

        tags = merge(
            var.tags,
            {
              Name = "Name of the resource"
            },
          )

The great thing about this is; anytime we need to make a change to the tags, we simply do that in one single place.

But, our key-value pairs are hard coded. So, go ahead and work out a fix for that. Simply create variables for each value and use var.variable_name as the value to each of the keys. 
Apply the same best practices for all other resources you are going to create.

- ![variableterraform](https://github.com/user-attachments/assets/0bf97a28-aa19-4a1e-9165-1661b8768720)
- ![mainterraform](https://github.com/user-attachments/assets/0128d06e-4044-41b9-9b54-5bba10c99267)



## Internet Gateways & format() function

Create an Internet Gateway in a separate Terraform file internet_gateway.tf

      resource "aws_internet_gateway" "ig" {
        vpc_id = aws_vpc.main.id
      
        tags = merge(
          var.tags,
          {
            Name = format("%s-%s!", aws_vpc.main.id,"IG")
          } 
        )
      }

Did you notice how we have used format() function to dynamically generate a unique name for this resource? The first part of the %s takes the interpolated value of aws_vpc.main.id while the second %s appends a literal string IG and finally an exclamation mark is added in the end.

If any of the resources being created is either using the count function or creating multiple resources using a loop, then a key-value pair that needs to be unique must be handled differently.

For example, each of our subnets should have a unique name in the tag section. Without the format() function, this would not be possible. With the format function, each private subnet's tag will look like this.


        Name = PrvateSubnet-0
        Name = PrvateSubnet-1
        Name = PrvateSubnet-2
Lets try and see that in action.

        tags = merge(
          var.tags,
          {
            Name = format("PrivateSubnet-%s", count.index)
          } 
        )

- ![privatesubnet0](https://github.com/user-attachments/assets/bd1aab0a-2df5-43aa-9a01-2a01d9c840a4)

- ![subnet1](https://github.com/user-attachments/assets/bc2883a5-2d10-45ad-9a1f-5ee87b7688f7)


## NAT Gateways
- Create 1 NAT Gateways and 1 Elastic IP (EIP) addresses

Now use similar approach to create the NAT Gateways in a new file called natgateway.tf.

Note: We need to create an Elastic IP for the NAT Gateway, and you can see the use of depends_on to indicate that the Internet Gateway resource must be available before this should be created.
Although Terraform does a good job to manage dependencies, in most cases, it is good to be explicit.

      resource "aws_eip" "nat_eip" {
        domain = "vpc"
        depends_on = [aws_internet_gateway.ig]
      
        tags = merge(
          var.tags,
          {
            Name = format("%s-EIP", var.name)
          },
        )
      }
      
      
      resource "aws_nat_gateway" "nat" {
        allocation_id = aws_eip.nat_eip.id
        subnet_id     = element(aws_subnet.public.*.id, 0)
        depends_on    = [aws_internet_gateway.ig]
      
        tags = merge(
          var.tags,
          {
            Name = format("%s-Nat", var.name)
          },
        )
      }

## AWS routes
Create a file called route_tables.tf and use it to create routes for both public and private subnets. Create the resources below and ensure they are properly tagged.

- aws_route_table
- aws_route
- aws_route_table_association

        # create private route table
        resource "aws_route_table" "private-rtb" {
          vpc_id = aws_vpc.main.id
        
          tags = merge(
            var.tags,
            {
              Name = format("%s-Private-Route-Table", var.name)
            },
          )
        }
        
        
        # associate all private subnets to the private route table
        resource "aws_route_table_association" "private-subnets-assoc" {
          count          = length(aws_subnet.private[*].id)
          subnet_id      = element(aws_subnet.private[*].id, count.index)
          route_table_id = aws_route_table.private-rtb.id
        }
        
        
        
        # create route table for the public subnets
        resource "aws_route_table" "public-rtb" {
          vpc_id = aws_vpc.main.id
        
          tags = merge(
            var.tags,
            {
              Name = format("%s-Public-Route-Table", var.name)
            },
          )
        }
        
        # create route for the public route table and attach the internet gateway
        resource "aws_route" "public-rtb-route" {
          route_table_id         = aws_route_table.public-rtb.id
          destination_cidr_block = "0.0.0.0/0"
          gateway_id             = aws_internet_gateway.ig.id
        }
        
        # associate all public subnets to the public route table
        resource "aws_route_table_association" "public-subnets-assoc" {
          count          = length(aws_subnet.public[*].id)
          subnet_id      = element(aws_subnet.public[*].id, count.index)
          route_table_id = aws_route_table.public-rtb.id
        }


Now if you run terraform plan and terraform apply, it will add the following resources to AWS in multi-az set up:

- [x] - Our main vpc
- [x] - 2 Public subnets
- [x] - 4 Private subnets
- [x] - 1 Internet Gateway
- [x] - 1 NAT Gateway
- [x] - 1 EIP
- [x] - 2 Route tables
Now that we are done with the Networking part of AWS set-up, let us move on to Compute and Access Control configuration automation using Terraform!

- ![EIP](https://github.com/user-attachments/assets/c17cf275-dfd7-4e6a-b80b-9fd3692ea901)

- ![NAT](https://github.com/user-attachments/assets/ea5c6aa7-d55f-4ac5-8dec-3cc6b1683263)

![Private1](https://github.com/user-attachments/assets/d4821040-b09a-498f-92b3-eddea90af220)

- ![private2](https://github.com/user-attachments/assets/becf37c7-ef87-4dbe-87e4-3a6299f433f4)

![private3](https://github.com/user-attachments/assets/88f8d64a-d25a-4fe0-b1f0-6af6408484b1)

![private4](https://github.com/user-attachments/assets/5ccd3b43-32a5-48a1-bc7c-cca2371c9652)

![public1](https://github.com/user-attachments/assets/0329ba9d-c3ad-4443-95b8-4ca4abef6573)

![public2](https://github.com/user-attachments/assets/06a4ce32-49f6-410c-a232-4b2fb7010e5c)

![vpcmain](https://github.com/user-attachments/assets/ed8fbb3b-f706-4e0d-93c4-bb002ace4381)

![igw](https://github.com/user-attachments/assets/85d9374f-22b6-450e-94ac-2d61bae12c09)

![route1](https://github.com/user-attachments/assets/9d401f36-40e8-43e9-b72e-5faafd8cdfac)

![routetableAssociastion](https://github.com/user-attachments/assets/e55fddf1-0090-4b57-b2c7-1020f09e980a)

## AWS Identity and Access Management
IaM and Roles
We want to pass an IAM role or EC2 instances to give them access to some specific resources, so we need to do the following:

### 1. Create AssumeRole
Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you may not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access.

Add the following code to a new file named roles.tf

      resource "aws_iam_role" "ec2_instance_role" {
      name = "ec2_instance_role"
        assume_role_policy = jsonencode({
          Version = "2012-10-17"
          Statement = [
            {
              Action = "sts:AssumeRole"
              Effect = "Allow"
              Sid    = ""
              Principal = {
                Service = "ec2.amazonaws.com"
              }
            },
          ]
        })
        
        tags = merge(
          var.tags,
          {
            Name = "aws assume role"
          },
        )
      }
In this code, we are creating _AssumeRole_ with _AssumeRole policy_. It grants to an entity, in our case it is an EC2, permissions to assume the role.

### 2. Create IAM policy for this role
This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform an action _describe_ applied to EC2 instances:

      resource "aws_iam_policy" "policy" {
        name        = "ec2_instance_policy"
        description = "A test policy"
        policy = jsonencode({
          Version = "2012-10-17"
          Statement = [
            {
              Action = [
                "ec2:Describe*",
              ]
              Effect   = "Allow"
              Resource = "*"
            },
          ]
      
        })
      
        tags = merge(
          var.tags,
          {
            Name =  "aws assume policy"
          },
        )
      
      }
### 3. Attach the Policy to the IAM Role
This is where, we will be attaching the policy which we created above, to the role we created in the first step.

     resource "aws_iam_role_policy_attachment" "test-attach" {
            role       = aws_iam_role.ec2_instance_role.name
            policy_arn = aws_iam_policy.policy.arn
        }

### 4. Create an Instance Profile and interpolate the IAM Role

          resource "aws_iam_instance_profile" "ip" {
              name = "aws_instance_profile_test"
              role =  aws_iam_role.ec2_instance_role.name
          }

We are pretty much done with the Identity and Management part for now, let us move on and create other resources required.

## Resources to be created
Going by our architecture, we need to do the following:

1. Create Security Groups.
2. Create Target Group for Nginx, WordPress and Tooling.
3. Create certificate from AWS certificate manager.
4. Create an External Application Load Balancer and Internal Application Load Balancer.
5. create launch template for Bastion, Tooling, Nginx and WordPress.
6. Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress.
7. Create Elastic Filesystem.
8. Create Relational Database (RDS)
   
Let us create some Terraform configuration code to accomplish these tasks.

## 1. Create Security Groups
We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it.

- Check out the terraform documentation for https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group
- Check out the terraform documentation for security group rule https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule

  
Create a file and name it security.tf, copy and paste the code below:

      # security group for alb, to allow acess from any where for HTTP and HTTPS traffic
      resource "aws_security_group" "ext-alb-sg" {
        name        = "ext-alb-sg"
        vpc_id      = aws_vpc.main.id
        description = "Allow TLS inbound traffic"
      
        ingress {
          description = "HTTP"
          from_port   = 80
          to_port     = 80
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
        ingress {
          description = "HTTPS"
          from_port   = 22
          to_port     = 22
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
       tags = merge(
          var.tags,
          {
            Name = "ext-alb-sg"
          },
        )
      
      }
      
      
      # security group for bastion, to allow access into the bastion host from you IP
      resource "aws_security_group" "bastion_sg" {
        name        = "vpc_web_sg"
        vpc_id = aws_vpc.main.id
        description = "Allow incoming HTTP connections."
      
        ingress {
          description = "SSH"
          from_port   = 22
          to_port     = 22
          protocol    = "tcp"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
         tags = merge(
          var.tags,
          {
            Name = "Bastion-SG"
          },
        )
      }
      
      
      
      #security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance
      resource "aws_security_group" "nginx-sg" {
        name   = "nginx-sg"
        vpc_id = aws_vpc.main.id
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
         tags = merge(
          var.tags,
          {
            Name = "nginx-SG"
          },
        )
      }
      
      resource "aws_security_group_rule" "inbound-nginx-http" {
        type                     = "ingress"
        from_port                = 443
        to_port                  = 443
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.ext-alb-sg.id
        security_group_id        = aws_security_group.nginx-sg.id
      }
      
      resource "aws_security_group_rule" "inbound-bastion-ssh" {
        type                     = "ingress"
        from_port                = 22
        to_port                  = 22
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.bastion_sg.id
        security_group_id        = aws_security_group.nginx-sg.id
      }
      
      
      # security group for ialb, to have access only from nginx reverser proxy server
      resource "aws_security_group" "int-alb-sg" {
        name   = "my-alb-sg"
        vpc_id = aws_vpc.main.id
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
        tags = merge(
          var.tags,
          {
            Name = "int-alb-sg"
          },
        )
      
      }
      
      resource "aws_security_group_rule" "inbound-ialb-https" {
        type                     = "ingress"
        from_port                = 443
        to_port                  = 443
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.nginx-sg.id
        security_group_id        = aws_security_group.int-alb-sg.id
      }
      
       
      # security group for webservers, to have access only from the internal load balancer and bastion instance
      resource "aws_security_group" "webserver-sg" {
        name   = "my-asg-sg"
        vpc_id = aws_vpc.main.id
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
        tags = merge(
          var.tags,
          {
            Name = "webserver-sg"
          },
        )
      
      }
      
      resource "aws_security_group_rule" "inbound-web-https" {
        type                     = "ingress"
        from_port                = 443
        to_port                  = 443
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.int-alb-sg.id
        security_group_id        = aws_security_group.webserver-sg.id
      }
      
      resource "aws_security_group_rule" "inbound-web-ssh" {
        type                     = "ingress"
        from_port                = 22
        to_port                  = 22
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.bastion_sg.id
        security_group_id        = aws_security_group.webserver-sg.id
      }
      
      
      # security group for datalayer to alow traffic from websever on nfs and mysql port and bastion host on mysql port
      resource "aws_security_group" "datalayer-sg" {
        name   = "datalayer-sg"
        vpc_id = aws_vpc.main.id
      
        egress {
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      
       tags = merge(
          var.tags,
          {
            Name = "datalayer-sg"
          },
        )
      }
      
      resource "aws_security_group_rule" "inbound-nfs-port" {
        type                     = "ingress"
        from_port                = 2049
        to_port                  = 2049
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.webserver-sg.id
        security_group_id        = aws_security_group.datalayer-sg.id
      }
      
      resource "aws_security_group_rule" "inbound-mysql-bastion" {
        type                     = "ingress"
        from_port                = 3306
        to_port                  = 3306
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.bastion_sg.id
        security_group_id        = aws_security_group.datalayer-sg.id
      }
      
      resource "aws_security_group_rule" "inbound-mysql-webserver" {
        type                     = "ingress"
        from_port                = 3306
        to_port                  = 3306
        protocol                 = "tcp"
        source_security_group_id = aws_security_group.webserver-sg.id
        security_group_id        = aws_security_group.datalayer-sg.id
      }


**IMPORTANT NOTE**: We used the aws_security_group_rule to reference another security group in a security group.

## 2. Create Certificate From Amazon Cerificate Manager
Create cert.tf file and add the following code snippets to it.

**NOTE**: Read through to change the domain name to your own domain name including any other name that needs to be changed.

The entire section create a certificate, public zone, and validate the certificate using DNS method. Create the certificate using a wildcard for all the domains created in _liberttinnii.xyz_.

- ![hostedZone](https://github.com/user-attachments/assets/36cf9c82-faaf-4d33-b205-3bcaef100fe2)




**NOTE**: You'd have to create a hosted zone from the AWS management console first before making reference to the created hosted zone in the code below;



          # The entire section creates a certificate, public zone, and validates the certificate using DNS method.
          
          # Create the certificate using a wildcard for all the domains created in oyindamola.gq
          resource "aws_acm_certificate" "oyindamola" {
            domain_name       = "*.oyindamola.gq"
            validation_method = "DNS"
          }
          
          # calling the hosted zone
          data "aws_route53_zone" "oyindamola" {
            name         = "oyindamola.gq"
            private_zone = false
          }
          
          # selecting validation method
          resource "aws_route53_record" "oyindamola" {
            for_each = {
              for dvo in aws_acm_certificate.oyindamola.domain_validation_options : dvo.domain_name => {
                name   = dvo.resource_record_name
                record = dvo.resource_record_value
                type   = dvo.resource_record_type
              }
            }
          
            allow_overwrite = true
            name            = each.value.name
            records         = [each.value.record]
            ttl             = 60
            type            = each.value.type
            zone_id         = data.aws_route53_zone.oyindamola.zone_id
          }
          
          # validate the certificate through DNS method
          resource "aws_acm_certificate_validation" "oyindamola" {
            certificate_arn         = aws_acm_certificate.oyindamola.arn
            validation_record_fqdns = [for record in aws_route53_record.oyindamola : record.fqdn]
          }
          
          # create records for tooling
          resource "aws_route53_record" "tooling" {
            zone_id = data.aws_route53_zone.oyindamola.zone_id
            name    = "tooling.oyindamola.gq"
            type    = "A"
          
            alias {
              name                   = aws_lb.ext-alb.dns_name
              zone_id                = aws_lb.ext-alb.zone_id
              evaluate_target_health = true
            }
          }
          
          
          # create records for wordpress
          resource "aws_route53_record" "wordpress" {
            zone_id = data.aws_route53_zone.oyindamola.zone_id
            name    = "wordpress.oyindamola.gq"
            type    = "A"
          
            alias {
              name                   = aws_lb.ext-alb.dns_name
              zone_id                = aws_lb.ext-alb.zone_id
              evaluate_target_health = true
            }
          }

## 3. Create an external (Internet facing) Application Load Balancer (ALB)
Create a file called _alb.tf_

First of all we will create the ALB, after which we create the target group and then, we create the lsitener rule.

We need to create an ALB to balance the traffic between the Instances:

          resource "aws_lb" "ext-alb" {
            name     = "ext-alb"
            internal = false
            security_groups = [
              aws_security_group.ext-alb-sg.id,
            ]
          
            subnets = [
              aws_subnet.public[0].id,
              aws_subnet.public[1].id
            ]
          
             tags = merge(
              var.tags,
              {
                Name = "ACS-ext-alb"
              },
            )
          
            ip_address_type    = "ipv4"
            load_balancer_type = "application"
          }

To inform our ALB where to route the traffic we need to create a _Target Group_ to point to its targets:

        resource "aws_lb_target_group" "nginx-tgt" {
          health_check {
            interval            = 10
            path                = "/healthstatus"
            protocol            = "HTTPS"
            timeout             = 5
            healthy_threshold   = 5
            unhealthy_threshold = 2
          }
          name        = "nginx-tgt"
          port        = 443
          protocol    = "HTTPS"
          target_type = "instance"
          vpc_id      = aws_vpc.main.id
        }

Then we will need to create a _Listner_ for this target group

      resource "aws_lb_listener" "nginx-listner" {
        load_balancer_arn = aws_lb.ext-alb.arn
        port              = 443
        protocol          = "HTTPS"
        certificate_arn   = aws_acm_certificate_validation.oyindamola.certificate_arn
      
        default_action {
          type             = "forward"
          target_group_arn = aws_lb_target_group.nginx-tgt.arn
        }
      }

Add the following outputs to _output.tf_ to print them on screen:

        output "alb_dns_name" {
          value = aws_lb.ext-alb.dns_name
        }
        
        output "alb_target_group_arn" {
          value = aws_lb_target_group.nginx-tgt.arn
        }

## Create an Internal (Internal) Application Load Balancer (ALB)
For the Internal Load balancer we will follow the same concepts as with the external load balancer.

Add the code snippets inside the alb.tf file

        # ----------------------------
        #Internal Load Balancers for webservers
        #---------------------------------
        
        resource "aws_lb" "ialb" {
          name     = "ialb"
          internal = true
          security_groups = [
            aws_security_group.int-alb-sg.id,
          ]
        
          subnets = [
            aws_subnet.private[0].id,
            aws_subnet.private[1].id
          ]
        
          tags = merge(
            var.tags,
            {
              Name = "ACS-int-alb"
            },
          )
        
          ip_address_type    = "ipv4"
          load_balancer_type = "application"
        }
To inform our ALB where to route the traffic, we need to create a _Target Group_ to point to its targets:

            # --- target group  for wordpress -------
            
            resource "aws_lb_target_group" "wordpress-tgt" {
              health_check {
                interval            = 10
                path                = "/healthstatus"
                protocol            = "HTTPS"
                timeout             = 5
                healthy_threshold   = 5
                unhealthy_threshold = 2
              }
            
              name        = "wordpress-tgt"
              port        = 443
              protocol    = "HTTPS"
              target_type = "instance"
              vpc_id      = aws_vpc.main.id
            }
            
            
            # --- target group for tooling -------
            
            resource "aws_lb_target_group" "tooling-tgt" {
              health_check {
                interval            = 10
                path                = "/healthstatus"
                protocol            = "HTTPS"
                timeout             = 5
                healthy_threshold   = 5
                unhealthy_threshold = 2
              }
            
              name        = "tooling-tgt"
              port        = 443
              protocol    = "HTTPS"
              target_type = "instance"
              vpc_id      = aws_vpc.main.id
            }

Then, we will need to create a _Listner_ for this target Group

          # For this aspect a single listener was created for the wordpress which is default,
          # A rule was created to route traffic to tooling when the host header changes
          
          
          resource "aws_lb_listener" "web-listener" {
            load_balancer_arn = aws_lb.ialb.arn
            port              = 443
            protocol          = "HTTPS"
            certificate_arn   = aws_acm_certificate_validation.oyindamola.certificate_arn
          
          
            default_action {
              type             = "forward"
              target_group_arn = aws_lb_target_group.wordpress-tgt.arn
            }
          }
          
          # listener rule for tooling target
          
          resource "aws_lb_listener_rule" "tooling-listener" {
            listener_arn = aws_lb_listener.web-listener.arn
            priority     = 99
          
            action {
              type             = "forward"
              target_group_arn = aws_lb_target_group.tooling-tgt.arn
            }
          
            condition {
              host_header {
                values = ["tooling.oyindamola.gq"]
              }
            }
          }

## Create an Auto Scaling Group (ASG)
Now, we need to configure our ASG to be able to scale the EC2s in and out, depending on the application traffic.

Before we start configuring an ASG, we need to create the launch template and the the AMI needed. For now we are going to use a random AMI from AWS, then in project 19, we will use Packerto create our ami.

Based on the architecture we need for Auto Scaling groups for bastion, nginx, wordpress and tooling, we will create two files; _asg-bastion-nginx.tf_ will contain Launch Template and Austoscaling group for Bastion and Nginx, while _asg-wordpress-tooling.tf_ will contain Launch Template and Austoscaling group for wordpress and tooling.

Create _asg-bastion-nginx.tf_ and paste the code below;

        # creating sns topic for all the auto scaling groups
        resource "aws_sns_topic" "david-sns" {
         name = "Default_CloudWatch_Alarms_Topic"
        }
        
        
        # creating notification for all the auto scaling groups
        resource "aws_autoscaling_notification" "david_notifications" {
         group_names = [
           aws_autoscaling_group.bastion-asg.name,
           aws_autoscaling_group.nginx-asg.name,
           aws_autoscaling_group.wordpress-asg.name,
           aws_autoscaling_group.tooling-asg.name,
         ]
         notifications = [
           "autoscaling:EC2_INSTANCE_LAUNCH",
           "autoscaling:EC2_INSTANCE_TERMINATE",
           "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
           "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
         ]
        
         topic_arn = aws_sns_topic.david-sns.arn
        }
        
        
        resource "random_shuffle" "az_list" {
         input        = data.aws_availability_zones.available.names
        }
        
        
        # launch template for bastion
        
        resource "aws_launch_template" "bastion-launch-template" {
         image_id               = var.ami
         instance_type          = "t2.micro"
         vpc_security_group_ids = [aws_security_group.bastion_sg.id]
        
         iam_instance_profile {
           name = aws_iam_instance_profile.ip.id
         }
        
         key_name = var.keypair
        
         placement {
           availability_zone = "random_shuffle.az_list.result"
         }
        
         lifecycle {
           create_before_destroy = true
         }
        
         tag_specifications {
           resource_type = "instance"
        
          tags = merge(
           var.tags,
           {
             Name = "bastion-launch-template"
           },
         )
         }
        
        
          # create a file called bastion.sh and copy the bastion userdata from project 15 into it
         user_data = filebase64("${path.module}/bastion.sh")
        }
        
        
        
        # ---- Autoscaling for bastion  hosts
        
        
        resource "aws_autoscaling_group" "bastion-asg" {
         name                      = "bastion-asg"
         max_size                  = 2
         min_size                  = 1
         health_check_grace_period = 300
         health_check_type         = "ELB"
         desired_capacity          = 1
        
         vpc_zone_identifier = [
           aws_subnet.public[0].id,
           aws_subnet.public[1].id
         ]
        
        
         launch_template {
           id      = aws_launch_template.bastion-launch-template.id
           version = "$Latest"
         }
         tag {
           key                 = "Name"
           value               = "bastion-launch-template"
           propagate_at_launch = true
         }
        
        }
        
        
        # launch template for nginx
        
        resource "aws_launch_template" "nginx-launch-template" {
         image_id               = var.ami
         instance_type          = "t2.micro"
         vpc_security_group_ids = [aws_security_group.nginx-sg.id]
        
         iam_instance_profile {
           name = aws_iam_instance_profile.ip.id
         }
        
         key_name =  var.keypair
        
         placement {
           availability_zone = "random_shuffle.az_list.result"
         }
        
         lifecycle {
           create_before_destroy = true
         }
        
         tag_specifications {
           resource_type = "instance"
        
           tags = merge(
           var.tags,
           {
             Name = "nginx-launch-template"
           },
         )
         }
        
          # create a file called nginx.sh and copy the nginx userdata from project 15 into it
         user_data = filebase64("${path.module}/nginx.sh")
        }
        
        
        # ------ Autoscslaling group for reverse proxy nginx ---------
        
        resource "aws_autoscaling_group" "nginx-asg" {
         name                      = "nginx-asg"
         max_size                  = 2
         min_size                  = 1
         health_check_grace_period = 300
         health_check_type         = "ELB"
         desired_capacity          = 1
        
         vpc_zone_identifier = [
           aws_subnet.public[0].id,
           aws_subnet.public[1].id
         ]
        
         launch_template {
           id      = aws_launch_template.nginx-launch-template.id
           version = "$Latest"
         }
        
         tag {
           key                 = "Name"
           value               = "nginx-launch-template"
           propagate_at_launch = true
         }
        
        
        }
        
        # attaching autoscaling group of nginx to external load balancer
        resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
         autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
         alb_target_group_arn   = aws_lb_target_group.nginx-tgt.arn
        }

Create _asg-wordpress-tooling.tf_ and paste the following code

            # launch template for wordpress
            
            resource "aws_launch_template" "wordpress-launch-template" {
              image_id               = var.ami
              instance_type          = "t2.micro"
              vpc_security_group_ids = [aws_security_group.webserver-sg.id]
            
              iam_instance_profile {
                name = aws_iam_instance_profile.ip.id
              }
            
              key_name = var.keypair
            
            
              placement {
                availability_zone = "random_shuffle.az_list.result"
              }
            
              lifecycle {
                create_before_destroy = true
              }
            
              tag_specifications {
                resource_type = "instance"
            
                tags = merge(
                var.tags,
                {
                  Name = "wordpress-launch-template"
                },
              )
            
              }
               
                # create a file called wordpress.sh and copy the wordpress userdata from project 15 into it.
              user_data = filebase64("${path.module}/wordpress.sh")
            }
            
            
            # ---- Autoscaling for wordpress application
            
            resource "aws_autoscaling_group" "wordpress-asg" {
              name                      = "wordpress-asg"
              max_size                  = 2
              min_size                  = 1
              health_check_grace_period = 300
              health_check_type         = "ELB"
              desired_capacity          = 1
              vpc_zone_identifier = [
            
                aws_subnet.private[0].id,
                aws_subnet.private[1].id
              ]
            
            
              launch_template {
                id      = aws_launch_template.wordpress-launch-template.id
                version = "$Latest"
              }
              tag {
                key                 = "Name"
                value               = "wordpress-asg"
                propagate_at_launch = true
              }
            }
            
            
            # attaching autoscaling group of wordpress application to internal loadbalancer
            resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
              autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
              alb_target_group_arn   = aws_lb_target_group.wordpress-tgt.arn
            }
            
            
            # launch template for tooling
            resource "aws_launch_template" "tooling-launch-template" {
              image_id               = var.ami
              instance_type          = "t2.micro"
              vpc_security_group_ids = [aws_security_group.webserver-sg.id]
            
              iam_instance_profile {
                name = aws_iam_instance_profile.ip.id
              }
            
              key_name = var.keypair
            
            
              placement {
                availability_zone = "random_shuffle.az_list.result"
              }
            
              lifecycle {
                create_before_destroy = true
              }
            
              tag_specifications {
                resource_type = "instance"
            
              tags = merge(
                var.tags,
                {
                  Name = "tooling-launch-template"
                },
              )
            
              }
              
              # create a file called tooling.sh and copy the tooling userdata from project 15 into it
              user_data = filebase64("${path.module}/tooling.sh")
            }
            
            
            
            # ---- Autoscaling for tooling -----
            
            resource "aws_autoscaling_group" "tooling-asg" {
              name                      = "tooling-asg"
              max_size                  = 2
              min_size                  = 1
              health_check_grace_period = 300
              health_check_type         = "ELB"
              desired_capacity          = 1
            
              vpc_zone_identifier = [
            
                aws_subnet.private[0].id,
                aws_subnet.private[1].id
              ]
            
              launch_template {
                id      = aws_launch_template.tooling-launch-template.id
                version = "$Latest"
              }
            
              tag {
                key                 = "Name"
                value               = "tooling-launch-template"
                propagate_at_launch = true
              }
            }
            
            # attaching autoscaling group of  tooling application to internal loadbalancer
            resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
              autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
              alb_target_group_arn   = aws_lb_target_group.tooling-tgt.arn
            }

## Storage and Database
Useful Terraform Documentation, go through this documentation and understand the arguement needed for each of the resources:

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_subnet_group
- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/efs_file_system
- https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kms_key
  
## Create Elastic File System (EFS)
In order to create an EFS you need to create a KMS key.

AWS Key Management Service (KMS) makes it easy for you to create and manage cryptographic keys and control their use across a wide range of AWS services and also in your applications.

Add the following code to efs.tf

        # create key from key management system
        resource "aws_kms_key" "ACS-kms" {
          description = "KMS key "
          policy      = <<EOF
          {
          "Version": "2012-10-17",
          "Id": "kms-key-policy",
          "Statement": [
            {
              "Sid": "Enable IAM User Permissions",
              "Effect": "Allow",
              "Principal": { "AWS": "arn:aws:iam::${var.account_no}:user/segun" },
              "Action": "kms:*",
              "Resource": "*"
            }
          ]
        }
        EOF
        }
        
        # create key alias
        resource "aws_kms_alias" "alias" {
          name          = "alias/kms"
          target_key_id = aws_kms_key.ACS-kms.key_id
        }


Let us create EFS and it's mount targets: Add the following code to efs.tf

        # create Elastic file system
        resource "aws_efs_file_system" "ACS-efs" {
          encrypted  = true
          kms_key_id = aws_kms_key.ACS-kms.arn
        
          tags = merge(
            var.tags,
            {
              Name = "ACS-efs"
            },
          )
        }
        
        
        
        # set first mount target for the EFS 
        resource "aws_efs_mount_target" "subnet-1" {
          file_system_id  = aws_efs_file_system.ACS-efs.id
          subnet_id       = aws_subnet.private[2].id
          security_groups = [aws_security_group.datalayer-sg.id]
        }
        
        
        # set second mount target for the EFS 
        resource "aws_efs_mount_target" "subnet-2" {
          file_system_id  = aws_efs_file_system.ACS-efs.id
          subnet_id       = aws_subnet.private[3].id
          security_groups = [aws_security_group.datalayer-sg.id]
        }
        
        
        # create access point for wordpress
        resource "aws_efs_access_point" "wordpress" {
          file_system_id = aws_efs_file_system.ACS-efs.id
        
          posix_user {
            gid = 0
            uid = 0
          }
        
          root_directory {
            path = "/wordpress"
        
            creation_info {
              owner_gid   = 0
              owner_uid   = 0
              permissions = 0755
            }
        
          }
        
        }
        
        
        # create access point for tooling
        resource "aws_efs_access_point" "tooling" {
          file_system_id = aws_efs_file_system.ACS-efs.id
          posix_user {
            gid = 0
            uid = 0
          }
        
          root_directory {
        
            path = "/tooling"
        
            creation_info {
              owner_gid   = 0
              owner_uid   = 0
              permissions = 0755
            }
        
          }
        }

## Create MySQL RDS
Let us create the RDS itself using this snippet of code in the _rds.tf_ file:

        # This section will create the subnet group for the RDS instance using the private subnet
        resource "aws_db_subnet_group" "ACS-rds" {
          name       = "acs-rds"
          subnet_ids = [aws_subnet.private[2].id, aws_subnet.private[3].id]
        
         tags = merge(
            var.tags,
            {
              Name = "ACS-rds"
            },
          )
        }
        
        
        # create the RDS instance with the subnets group
        resource "aws_db_instance" "ACS-rds" {
          allocated_storage      = 20
          storage_type           = "gp2"
          engine                 = "mysql"
          engine_version         = "5.7"
          instance_class         = "db.t2.micro"
          name                   = "daviddb"
          username               = var.master-username
          password               = var.master-password
          parameter_group_name   = "default.mysql5.7"
          db_subnet_group_name   = aws_db_subnet_group.ACS-rds.name
          skip_final_snapshot    = true
          vpc_security_group_ids = [aws_security_group.datalayer-sg.id]
          multi_az               = "true"
        }

Before applying, please note that we gave reference to some variables in our resources that have not been declared in the _variables.tf_ file. Go through the entire code and spot these variables and declare them in the _variables.tf_ file.

If you have done that well, your file should look like this one below.




            variable "region" {
              type = string
              description = "The region to deploy resources"
            }
            
            variable "vpc_cidr" {
              type = string
              description = "The VPC cidr"
            }
            
            variable "enable_dns_support" {
              type = bool
            }
            
            variable "enable_dns_hostnames" {
              dtype = bool
            }
            
            variable "enable_classiclink" {
              type = bool
            }
            
            variable "enable_classiclink_dns_support" {
              type = bool
            }
            
            variable "preferred_number_of_public_subnets" {
              type        = number
              description = "Number of public subnets"
            }
            
            variable "preferred_number_of_private_subnets" {
              type        = number
              description = "Number of private subnets"
            }
            
            variable "name" {
              type    = string
              default = "ACS"
            
            }
            
            variable "tags" {
              description = "A mapping of tags to assign to all resources."
              type        = map(string)
              default     = {}
            }
            
            
            variable "ami" {
              type        = string
              description = "AMI ID for the launch template"
            }
            
            
            variable "keypair" {
              type        = string
              description = "key pair for the instances"
            }
            
            variable "account_no" {
              type        = number
              description = "the account number"
            }
            
            
            variable "master-username" {
              type        = string
              description = "RDS admin username"
            }
            
            variable "master-password" {
              type        = string
              description = "RDS master password"
            }

We are almost done but we need to update the last file which is _terraform.tfvars_ file. In this file we are going to declare the values for the variables in our _varibales.tf_ file.

Open the _terraform.tfvars_ file and add the code below:

        region = "us-east-1"
        
        vpc_cidr = "172.16.0.0/16"
        
        enable_dns_support = "true"
        
        enable_dns_hostnames = "true"
        
        enable_classiclink = "false"
        
        enable_classiclink_dns_support = "false"
        
        preferred_number_of_public_subnets = "2"
        
        preferred_number_of_private_subnets = "4"
        
        environment = "production"
        
        ami = "ami-0b0af3577fe5e3532"
        
        keypair = "devops"
        
        # Ensure to change this to your acccount number
        account_no = "123456789"
        
        
        db-username = "david"
        
        
        db-password = "devopspbl"
        
        
        tags = {
          Enviroment      = "production" 
          Owner-Email     = "infradev-segun@darey.io"
          Managed-By      = "Terraform"
          Billing-Account = "1234567890"
        }

At this point, you shall have pretty much all the infrastructure elements ready to be deployed automatically. Try to plan and apply your Terraform codes, explore the resources in AWS console and make sure you destroy them right away to avoid massive costs.

- ![created](https://github.com/user-attachments/assets/111eae27-6e4f-42b6-b040-1e2c2da3b5b0)

- ![Tapply1](https://github.com/user-attachments/assets/a09a2d2a-c83d-49e6-96a8-7a0da1f7e04f)

- ![Tapply2](https://github.com/user-attachments/assets/00b9d580-1deb-4f9e-a2b2-ffa91d9c4d3c)

- ![Tapply3](https://github.com/user-attachments/assets/09f2867c-e14f-4f67-af07-5e5f9c8315cf)

- ![Tapply4](https://github.com/user-attachments/assets/29299c2c-f1c1-4a26-8fe3-18f9412c53a7)

- ![Tapply5](https://github.com/user-attachments/assets/773a75da-7177-4b66-bfb9-7c05d9c33382)

- ![Tapply6](https://github.com/user-attachments/assets/b6041d91-765e-4cbd-8ed2-2c8077f76d49)

- ![Tapply7](https://github.com/user-attachments/assets/530e10c4-6b99-4ff2-aa4d-c6134e2b68ce)

- ![Tapply8](https://github.com/user-attachments/assets/604adeee-3c17-4be9-8a2e-f6381a3ba64b)

- ![Tapply9](https://github.com/user-attachments/assets/3a8602c0-0020-4130-bca1-7c162adf43bc)

- ![Tapply10](https://github.com/user-attachments/assets/b84627aa-b821-4aa7-8ae9-0ffc38c5bc00)

- ![Tapply11](https://github.com/user-attachments/assets/e9e1beab-e97a-4b1c-a98f-032e37447e8a)

- ![Tapply12](https://github.com/user-attachments/assets/0afab8b9-522f-4d86-b5b7-da6fa28a0567)

- ![Tapply13](https://github.com/user-attachments/assets/0f6f8179-0e82-4141-9375-bb33cbf583c9)

- ![Tapply14](https://github.com/user-attachments/assets/3c22371c-6b6f-4215-8296-8ff695e6f5f6)


- ![destroy](https://github.com/user-attachments/assets/b5bbcf5a-fab8-4cc0-9797-3a549cb4ed27)








































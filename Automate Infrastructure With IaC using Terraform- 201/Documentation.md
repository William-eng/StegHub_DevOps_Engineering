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




















































































































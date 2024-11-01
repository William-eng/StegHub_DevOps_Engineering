# Automate Infrastructure With IaC using Terraform 3 (Refactoring)- 301

In two previous projects we have developed AWS Infrastructure code using Terraform and tried to run it from your local workstation. Now it is time to introduce some more advanced concepts and enhance our code.

Firstly, we will explore alternative Terraform backends.

## Introducing Backend on S3

Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc. Take a peek into what the states file looks like. It is basically where terraform stores all the state of the infrastructure in _json_ format.

So far, we have been using the default backend, which is the _local backend_ - it requires no configuration, and the states file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.

The second problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on your computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely other DevOps team members. There are plenty of different standard backends supported by Terraform that you can choose from. Since we are already using AWS - we can choose an S3 bucket as a backend.

Another useful option that is supported by S3 backend is State Locking - it is used to lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service - DynamoDB.

Let us configure it!

Here is our plan to Re-initialize Terraform to use S3 backend:

- Add S3 and DynamoDB resource blocks before deleting the local state file
- Update terraform block to introduce backend and locking
- Re-initialize terraform
- Delete the local tfstate file and check the one in S3 bucket
- Add _outputs_
  
_terraform apply_

1. Create a file and name it _backend.tf_. Add the below code and replace the name of the S3 bucket you created in previous project.

        # Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.
        resource "aws_s3_bucket" "terraform_state" {
            bucket = "Ktrontech-terraform"
          
            # Add tags if needed
            tags = {
              Name = "Ktrontech-terraform-state"
            }
          }
          
          
          resource "aws_s3_bucket_versioning" "terraform_state_versioning" {
            bucket = aws_s3_bucket.terraform_state.bucket
          
            versioning_configuration {
              status = "Enabled"
            }
          }
          
          resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state_encryption" {
            bucket = aws_s3_bucket.terraform_state.bucket
          
            rule {
              apply_server_side_encryption_by_default {
                sse_algorithm = "AES256"
              }
            }
          }



You must be aware that Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there. Hence, you must consider to always 
enable encryption. You can see how we achieved that with _server_side_encryption_configuration_.

2. Next, we will create a DynamoDB table to handle locks and perform consistency checks. In previous projects, locks were handled with a local file as shown in terraform.tfstate.lock.info. 
Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore, with a cloud storage database like DynamoDB,
anyone running Terraform against the same infrastructure can use a central location to control a situation where Terraform is running at the same time from multiple different people.

Dynamo DB resource for locking and consistency checking:

      resource "aws_dynamodb_table" "terraform_locks" {
        name         = "terraform-locks"
        billing_mode = "PAY_PER_REQUEST"
        hash_key     = "LockID"
        attribute {
          name = "LockID"
          type = "S"
        }
      }
- ![backend](https://github.com/user-attachments/assets/6028ca08-f246-45a6-8154-907dbfebaa00)

Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run _terraform apply_ to provision resources.

3. Configure S3 Backend

          terraform {
            backend "s3" {
              bucket         = "dev-terraform-bucket"
              key            = "global/s3/terraform.tfstate"
              region         = "eu-central-1"
              dynamodb_table = "terraform-locks"
              encrypt        = true
            }
          }

Now its time to re-initialize the backend. Run _terraform init_ and confirm you are happy to change the backend by typing _yes_

Verify the changes
Before doing anything if you opened AWS now to see what happened you should be able to see the following:

- _.tfstatefile_ is now inside the S3 bucket

- ![statefile](https://github.com/user-attachments/assets/30cfd32a-e7fd-4fef-96d6-c947668c4098)

DynamoDB table which we create has an entry which includes state file status

- ![DynamoDB](https://github.com/user-attachments/assets/8c40cfc4-10e5-4f79-b0a3-d4eddc397d96)


Navigate to the DynamoDB table inside AWS and leave the page open in your browser. Run terraform plan and while that is running, refresh the browser and see how the lock is being handled:

- ![DBrun](https://github.com/user-attachments/assets/290567a1-5657-46dd-bea5-ce8f459ff612)

After _terraform plan_ completes, refresh DynamoDB table.

- ![plannn](https://github.com/user-attachments/assets/e2e885f4-52d7-42e9-8daf-35c4437d6ece)

5. Add Terraform Output
Before you run _terraform apply_ let us add an output so that the S3 bucket Amazon Resource Names ARN and DynamoDB table name can be displayed.

Create a new file and name it output.tf and add below code.

        output "s3_bucket_arn" {
          value       = aws_s3_bucket.terraform_state.arn
          description = "The ARN of the S3 bucket"
        }
        output "dynamodb_table_name" {
          value       = aws_dynamodb_table.terraform_locks.name
          description = "The name of the DynamoDB table"
        }


6. Let us run _terraform apply_
Terraform will automatically read the latest state from the S3 bucket to determine the current state of the infrastructure. Even if another engineer has applied changes, the state file will always be up to date.

Now, head over to the S3 console again, refresh the page, and click the grey “Show” button next to “Versions.” You should now see several versions of your terraform.tfstate file in the S3 bucket:

- ![Version1](https://github.com/user-attachments/assets/dd14d254-cdea-4226-afea-2756caa19eb0)

With help of remote backend and locking configuration that we have just configured, collaboration is no longer a problem.

However, there is still one more problem: **Isolation Of Environments**. Most likely we will need to create resources for different environments, such as: _Dev, sit, uat, preprod, prod,_ etc.

This separation of environments can be achieved using one of two methods:

a. **Terraform Workspaces** b. **Directory** based separation using terraform.tfvars file

## When to use Workspaces or Directory?
To separate environments with significant configuration differences, use a directory structure. Use workspaces for environments that do not greatly deviate from each other, to avoid duplication of your configurations. Try both methods in the sections below to help you understand which will serve your infrastructure best.

## Security Groups refactoring with dynamic block

For repetitive blocks of code you can use dynamic blocks in Terraform,

Refactor Security Groups creation with dynamic blocks.

we can have a local.tf like this 

      locals {
        ingress_rules = [
          {
            description = "HTTP"
            port        = 80
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            description = "SSH"
            port        = 22
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          }
        ]
      
        egress_rules = [
          {
            port        = 0
            protocol    = "-1"
            cidr_blocks = ["0.0.0.0/0"]
          }
        ]
      }
and we can incorporate this in our security group rule

      resource "aws_security_group" "example" {
        name        = "example-security-group"
        description = "Security group with dynamic ingress and egress rules"
        vpc_id      = "your-vpc-id"
      
        dynamic "ingress" {
          for_each = local.ingress_rules
          content {
            description = ingress.value.description
            from_port   = ingress.value.port
            to_port     = ingress.value.port
            protocol    = ingress.value.protocol
            cidr_blocks = ingress.value.cidr_blocks
          }
        }
      
        dynamic "egress" {
          for_each = local.egress_rules
          content {
            from_port   = egress.value.port
            to_port     = egress.value.port
            protocol    = egress.value.protocol
            cidr_blocks = egress.value.cidr_blocks
          }
        }
      }



## EC2 refactoring with Map and Lookup

Remember, every piece of work you do, always try to make it dynamic to accommodate future changes. Amazon Machine Image (AMI) is a regional service which means it is only available in the region it was created. But what if we change the region later, and want to dynamically pick up AMI IDs based on the available AMIs in that region? This is where we will introduce Map and Lookup functions.

Map uses a key and value pairs as a data structure that can be set as a default type for variables.

      variable "images" {
          type = "map"
          default = {
              us-east-1 = "image-1234"
              us-west-2 = "image-23834"
          }
      }

To select an appropriate AMI per region, we will use a lookup function which has following syntax: _lookup(map, key, [default])_.

**Note**: A default value is better to be used to avoid failure whenever the map data has no key.

      resource "aws_instace" "web" {
          ami  = "${lookup(var.images, var.region), "ami-12323"}
      }


Now, the lookup function will load the variable images using the first parameter. But it also needs to know which of the key-value pairs to use. That is where the second parameter comes in. The key us-east-1 could be specified, but then we will not be doing anything dynamic there, but if we specify the variable for region, it simply resolves to one of the keys. That is why we have used _var.region_ in the second parameter.

























































































































































































































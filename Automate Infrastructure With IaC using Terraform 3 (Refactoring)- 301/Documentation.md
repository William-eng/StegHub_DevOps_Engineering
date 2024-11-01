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

- 
DynamoDB table which we create has an entry which includes state file status

- 

Navigate to the DynamoDB table inside AWS and leave the page open in your browser. Run terraform plan and while that is running, refresh the browser and see how the lock is being handled:
























































































































































































































































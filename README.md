## AWS Cloud Engineer

**Greetings! We'll be going over the tasks for Week 6 - Days 2 of the Cloud Engineering program. Lets get started!**

## Week 6 - Day 2: Action Steps
* Install Terraform
* Log into the AWS console
* Create Terraform code to create an EC2 instance
* Create modules for EC2 instance and S3 bucket
* Input terraform init, plan, apply and destroy
________________________

## Install Terraform

Welcome back! 

Now we're going to be going over A TON of information for this lab. I'm familiar enough with Terraform from Azure. I've used it to deploy an entire public load balancer infrastructure with a NAT Gateway, Azure Bastion, and jumpbox virtual machine (VM). I used variables and output files as well. I never actually made a module before so that process was new to me. 

So with that being said, I needed to get familiar with AWS' Terraform environment. Before we even do that, we need to install Terraform on our local machine. But even before that, we need to install the AWS Cloud Shell into our local machine as well so lets get started with those things. We'll begin with the AWS download. Go to Google and enter AWS Windows Installer. 

![Install AWS Cloud Shell](OnePercentWeek6Day2_Task1.png)

Copy and paste the installer command in Command Prompt (CMD) to install the AWS Bash CLI. Restart CMD or VS Code and then run `aws --version` to confirm the CLI is installed. 

![Verifies AWS has been installed](OnePercentWeek5Day1_Task2.png)

Lets do the same with Terraform which was a little more involved. Go to the website and download your respective installer (I'm on a Windows device so I chose AMD64).


![Install Terraform](OnePercentWeek5Day1_Task3.png)

Now, you need to take your `terraform.exe` file and create a path for it. I was instructed to create a separate folder for Terraform (C:\Terraform\). Follow the instructions below and save your new path. 

![Environment path for Terraform](OnePercentWeek5Day1_Task4.png)

Once you restart CMD, you can run `terraform -version` to make sure it's installed. 

![Terraform verification](OnePercentWeek5Day1_Task5.png)

Way too much work but wait, there's more! Lets move to configuring the AWS console so we can actually connect our Terraform commands to AWS.

## Log into the AWS console

Now, lets connect to AWS. We can use `aws configure` to input our credentials into the CLI to connect to our AWS account. I cant run the commands again but it will ask for the AWS Access Key ID, Secret Access Key, default region name and default output. You can find this information by going to your AWS console, clicking your profile in the upper-right corner, cliking Security Credentials, and then scroll down to Access keys. You can generate your keys for aws configure. I put in my default region and I chose json for my output. 

Once you're done, use `aws sts get-caller-identity` in order to see that you're logged in. 

![Verify AWS login](OnePercentWeek5Day1_Task6.png)

Alright, NOW we can finally get over to the Terraform part of the lab!


## Create Terraform code to create EC2 instance and S3

Before we start up Terraform, lets go over the main files for Terraform: providers.tf and main.tf.

We'll start with the providers.tf file. It tells us which providers (whether Azure, AWS, or even Docker) we'll be using to construct our infrastructure. It also tells us which Terraform (TF) version we should be using and for AWS, it asks us for our region. This is great because Azure asks for the region for almost all the resources so this makes things easier for us to code. 

![providers.tf file](OnePercentWeek5Day1_Task7.png)

The main.tf file is where the meat and potatoes are. This is where we define our resources that we want to be created. It usually has the format resource "name_of_resource" "resource_arbitrary_name" {}. Then, everything inside the bracket is what we need to code to define our resource. Each resource has mandatory values and optional values. You use that arbitrary name to call upon THAT particular resource. You might need a parameter of the resource so you use that name to specifically call those parameters. 

![providers.tf file](OnePercentWeek5Day1_Task8.png)

Lastly, we have the optional outputs.tf file. This is where we tell TF what information we want to see after it's done creating the resources. I want the IP addresses of my EC2 instances and the name of the my S3 bucket so that goes into this file. We also have a variables.tf file which is where we make names and default values for variables that we can use throughout our files. We call upon those variables using the format `var.<variable_name>`.  This makes called on strings and values WAY easier to remember. This was extremely useful in Azure. Not as much in this AWS lab. 

Task completed!

## Create Terraform code to create an EC2 instance Pt. 2

Now, we need to create an EC2 instance using TF. Our next lab which is based on Ansible wants us to create a master node an 3 worker nodes so I went ahead and created the modules for our VPC, Security Group (SG), EC2 instances, and S3 bucket just to make that process easier. So lets jump straight into the code for the EC2 instance. I'll explain the module and directory tree structure later. 

We have to define the AWS resource, AMI, which subnet it belongs to, the side of the instacce, the security group, and our key pair that we'll use to SSH into the machine. 

![EC2 instance main.tf](OnePercentWeek5Day1_Task9.png)

I did create a variables.tf file to pass along the subnet ID, name, public key, and SG to the instance. I chose the instance ID and IP address for the outputs. That's all I really need. 

If I run my code, it should create the EC2 instance!

Lets move onto the modules and tree structure. 


## Create modules for EC2 instance and S3 bucket

Now, this section will be pretty long but I have to explain my logic behind the entire process. Okay, what exactly is a module? A module in TF is a reusable block of code which will make your code more readable and reproducible. TF is already reusable but think of you creating a VM for your company with specific parameters that you need each VM to have. You can create this template for the VM and reuse it in different environments in your enterprise. You can also feed this module specific parameters that you can change from resource to resource while keeping the base the same. 

This considerably shrinks your main.tf file and gives you more flexibility while increasin reproducibilty. Its a win for everybody!

Lets talk file structure now. You have to create a main.tf file for each one of your modules. You may need to create a variables.tf and outputs.tf file for each module as well. You'll have an overall (or root) main.tf, providers.tf, variables.tf, and outputs.tf file but within that same directory, you'll have a modules directory. under that directory, you create all of your module names. Mine were network, s3, security_group, and ec2_instance. Then you create a main.tf file under those directories. 


![tree structure](OnePercentWeek5Day1_Task10.png)

I would advise using a mix of mkdir -p and the brackets {} to create all of the files and folders pretty quickly instead of manually typing this information out for each module. for example, you can perhaps do 

`touch ~/terraform-aws/{main,outputs,providers,variables}.tf`

and

`mkdir -p ~/terraform-aws/modules/{network,ec2_instance,s3,security_group}`

then `touch ~/terraform-aws/modules/{network,ec2_instance,s3,security_group}/{main,variables.outputs}.tf`

PHEW! Alright, now just like in a regular main.tf file, you go and configure your resources in your respective module's main.tf file. For networking, I put all of my VPC resources here. Overall, I created the VPC, public subnet, internet gateway, route table, default route, and associated the route table to the public subnet. 

Etc, etc. Lets look at the structure of the modules in the root main.tf file. 

![Module structure](OnePercentWeek5Day1_Task11.png)

I chose to look at our EC2 module because you see we were able to name our module specifically and call the source of where we're getting our module from. The source is the general module format we created and then in this specifically named module, we can feed our general module custom information. 

I did a module for our master node which I passed it an SSH public key. For my worker node module, I gave my module count = 3 so it would create 3 of these VMs.

How useful is that!


Now I want to comment on the root output.tf file because this was a bit confusing at first. 

![Main outputs.tf explanation](OnePercentWeek5Day1_Task12.png)

Since the EC2 module itself has outputs, how do we call specifically the output for each specific EC2 module we create? We have to call the module, then module specific name, and THEN the output that we were looking for. So we're always able to get the information that we want. 

Now, for the fun part. Lets actually deploy our Infrastructure as Code (IaC)!

## Input terraform init, plan, apply and destroy

First, we have to get TF running so run the command `terraform init`. 

![terraform init](OnePercentWeek5Day1_Task13.png)

You should get a confirmation message from this. Now lets use `terraform validate` and `terraform plan` to make sure our syntax is good and confirm that TF knows what it will be creating. 

![terraform validate](OnePercentWeek5Day1_Task14.png)

![terraform plan](OnePercentWeek5Day1_Task15.png)

Now, lets actually deploy it all. Run `terraform apply --auto-approve` and sit back. 

![terraform apply](OnePercentWeek5Day1_Task16.png)

You can verify in the AWS console that your resources have been created. Also, lets SSH into our master node to make sure the EC2 instance actually works. 

![AWS Console Resource Verification](OnePercentWeek5Day1_Task17.png)


![Master node SSH verification](OnePercentWeek5Day1_Task18.png)

Super useful. Now that we know everything works, run `terraform destroy --auto-approve` to delete everything. 

![terraform destroy](OnePercentWeek5Day1_Task19.png)

Then confirm in the AWS console that the resources are no longer there. 

![AWS Console Resource Deletion Verification](OnePercentWeek5Day1_Task20.png)

Terraform is so useful! Hopefully you're a believer. We're going to use this same reusable IaC to run our Ansible lab next. 

Stay tuned!

## Personal Notes

AWS was so much easier to learn TF on than Azure. The modules were extremely useful and I can see exactly how people deploy both Infrastructure and Application pipelines because I know you can use Docker with TF as well. 

The thing that gave me the most issues with was the SSH keys. Now, I'm familiar with SSH connections only because of the RHCSA. I know the networking side of it but not really the actual files and everything that goes with that. So I did use Chat GPT to help me structure the key part of the root main.tf file because it's just new to me. 

I needed to generate the crypto file, make a private key, automatically download that private key to my local machine, automatically pass the public key to the EC2 instances of my choice, and...I think that's it. These resource blocks weren't even labeled AWS so that was the real OS side of TF that I've never used before. I had to google what each part meant to make sure I fully understoof it. 

I'll talk about the next part of the SSH issue in my Ansible lab but yeah. The problems didn't stop there. Eventually it all came together though. I learned some useful things here so I'm grateful!
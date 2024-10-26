### Create Terraform Repository

To create a Terraform repository from the terminal, follow these steps:

### Step 1: Install Git and Terraform
Ensure Git and Terraform are installed on your system.

- **Install Git**:
  ```sh
  sudo apt-get update
  sudo apt-get install git
  ```

- **Install Terraform**:
  Download Terraform from the official [Terraform download page](https://www.terraform.io/downloads.html) and follow the instructions for your operating system.

### Step 2: Set Up GitHub Repository
Create a new repository on GitHub. You can do this from the GitHub website by navigating to your profile and selecting **Repositories > New**. Name your repository and create it.

### Step 3: Clone the Repository Locally
Clone the newly created repository to your local machine.

```sh
git clone https://github.com/your-username/your-repository.git
cd your-repository
```

### Step 4: Initialize Terraform
Initialize your Terraform project.

1. Create a new directory for your Terraform files if you haven't already:

    ```sh
    mkdir terraform
    cd terraform
    ```

2. Initialize Terraform:

    ```sh
    terraform init
    ```

### Step 5: Create Terraform Configuration Files
Create the necessary Terraform configuration files (`.tf` files).

1. **Main Configuration File (`main.tf`)**:

    ```sh
    touch main.tf
    ```

    Add your Terraform configuration to `main.tf`. For example:

    ```hcl
    provider "aws" {
      region = "us-west-2"
    }

    resource "aws_instance" "example" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.micro"

      tags = {
        Name = "example-instance"
      }
    }
    ```

2. **Variables File (`variables.tf`)** (optional):

    ```sh
    touch variables.tf
    ```

    Define variables in `variables.tf`. For example:

    ```hcl
    variable "instance_type" {
      description = "Type of instance to create"
      default     = "t2.micro"
    }
    ```

3. **Outputs File (`outputs.tf`)** (optional):

    ```sh
    touch outputs.tf
    ```

    Define outputs in `outputs.tf`. For example:

    ```hcl
    output "instance_id" {
      value = aws_instance.example.id
    }
    ```

### Step 6: Add and Commit Your Changes
Add and commit your Terraform configuration files to the Git repository.

```sh
git add .
git commit -m "Initial commit with Terraform configuration"
```

### Step 7: Push to GitHub
Push your changes to the GitHub repository.

```sh
git push origin main
```

### Step 8: Running Terraform Plan and Apply
From the terminal, run `terraform plan` and `terraform apply` to see the execution plan and apply the configuration.

1. **Terraform Plan**:
    ```sh
    terraform plan
    ```

2. **Terraform Apply**:
    ```sh
    terraform apply
    ```

Follow the prompts to confirm the actions.

### Summary
These steps outline how to set up a Terraform project and repository from the terminal. Ensure your AWS credentials or any other provider credentials are correctly configured in your environment for Terraform to authenticate and execute the commands.

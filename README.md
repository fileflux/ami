
# AMI Builder Using Packer

## Overview
This project builds an Amazon Machine Image (AMI) using [Packer](https://www.packer.io/). Packer automates the creation of machine images across multiple platforms, enabling consistent and reproducible AMIs for deployment on AWS. 

In this project, shell scripts are used in conjunction with Packer to initialize, configure, and finalize the creation of an AMI. Additionally, AWS CLI and `jq` are used for managing AWS resources and processing JSON data, respectively.

## Prerequisites

To build and run this project, you'll need to install the following:

- **Packer**: Install it from [here](https://www.packer.io/docs/install/index.html).
- **AWS CLI**: The AWS Command Line Interface can be installed following this [guide](https://docs.aws.amazon.com/cli/latest/userguide/installing.html).
- **jq**: A lightweight and flexible command-line JSON processor. Install it from [here](https://stedolan.github.io/jq/download/).

Make sure your AWS CLI is properly configured with the necessary credentials and permissions to create and manage AMIs in your AWS account.

## Repository Structure

```plaintext
ami/
├── ami.pkr.hcl            
├── init.sh                
├── heartbeat.sh           
├── finish.sh              
├── zfs_add_script.sh      
├── zfs_pool_init.sh       
├── .git/                  
└── README.md              
```

### What Each File Does

- **ami.pkr.hcl**: The main Packer configuration file that defines how to build the AMI. This file contains instructions for provisioning, installing dependencies, and setting up the environment.
  
- **init.sh**: This script sets up the required environment before building the AMI. It includes upgrading the base OS with the latest packages and installing zfsutils-linux for ZFS support.
  
- **heartbeat.sh**: A script to be part of the AMI, allowing the EKS worker node to send heartbeats with information such as the node name, ZFS pool name, total pool size, and available pool size to the EKS cluster. 
  
- **finish.sh**: Final stage of the build process which moves relavent files to the correct locations and creates necessary services for various scripts.
  
- **zfs_add_script.sh**: A script to create additional EBS volumes on the self-managed EKS node and add it to the existing ZFS pool, to increase the storage pool size. Pairs with the Ansible playbook deploy.yaml in the FileFlux Ansible repository.
  
- **zfs_pool_init.sh**: Initializes the ZFS pools, which are storage constructs in the ZFS filesystem, ensuring that the instance is ready to use them upon boot.

## AMI Creation Process

1. **Initialize the environment**: 
   The `init.sh` script sets up the necessary prerequisites for the build environment, including any dependencies or configuration.

2. **Run Packer**: 
   Packer, based on the configuration file `ami.pkr.hcl`, provisions a temporary instance on AWS, installs the necessary software, and runs the provided scripts to configure the instance.

3. **Use scripts**: 
   The various scripts (`heartbeat.sh`, `finish.sh`, `zfs_add_script.sh`, etc.) are executed/copied at different stages to ensure the instance is properly configured.

4. **Build output**: 
   Once the build process is completed successfully, Packer will create an AMI in your specified AWS region, which can then be used to launch new EC2 instances.

## Usage

1. **Clone the repository**: 
   First, clone the repository to your local machine.
   ```bash
   git clone https://github.com/fileflux/ami.git
   cd ami
   ```

2. **Install the required tools**: 
   Ensure that `packer`, `awscli`, and `jq` are installed and accessible via your command line.

3. **Configure AWS CLI**:
   Run the following command to configure your AWS credentials:
   ```bash
   aws configure
   ```

4. **Run Packer**:
   - You can build the AMI by running the following command:
     ```bash
     packer build ami.pkr.hcl
     ```
   - Packer will use the provided `ami.pkr.hcl` configuration file to create an AMI based on the defined settings and scripts in this project.

5. **Monitor Build Process**:
   During the Packer build process, logs will be displayed that outline each step. If successful, Packer will output the AMI ID at the end of the process, which can then be used for launching instances on AWS.

## Notes

- Make sure to have the necessary IAM permissions to create AMIs, launch EC2 instances, and modify EC2 configurations in your AWS account.
- Adjust the `ami.pkr.hcl` file to your specific needs (e.g., region, instance type).

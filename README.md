# Dashboard_demo_2

The repository contains the instructions and code that allows you to create a kubernetes cluster for dashboard demo using virtual can interface

## THIS DEMO WILL WORK BEST ON MUMBAI (ap-south-1) region

Deploy EC2 in Mumbai region

[![Launch](https://samdengler.github.io/cloudformation-launch-stack-button-svg/images/ap-south-1.svg)](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/quickcreate?templateURL=https%3A%2F%2Fs3.ap-south-1.amazonaws.com%2Fcf-templates-fui01m96flo3-ap-south-1%2F2023-06-16T050256.331Zcy7-demo-2-dashboard-ec2-template.yml&stackName=dashboard-demo-2-SDV-stack)

Acknowledge the creation of the stack and press the button **Create stack** on the bottom right. 

The ```dashboard-demo-2-SDV-stack``` CloudFormation stack will take about **2 minutes** to be created.

### Locally downloading the Private key file

Follow the steps below to download the private .pem key file to SSH into the instance

Open cloudshell and run the following command

```sh 
aws ec2 describe-key-pairs --filters Name=key-name,Values=keypair-for-ewaol --query KeyPairs[*].KeyPairId --output text
```

The output will be the key ID. Note down it

Run the below command to save .pem file in the cloudshell directory

Change the keyid paramater to the output of previous command

```sh
aws ssm get-parameter --name /ec2/keypair/<keyid here> --with-decryption --query Parameter.Value --output text > keypair-for-ewaol.pem
```

<img width="911" alt="Screenshot 2023-06-16 085413" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/502fba9f-99d6-4a10-a806-df7239149409">

Go to actions -> download file and paste this path "/home/cloudshell-user/keypair-for-ewaol.pem" inside the path field to save .pem key file locally

<img width="516" alt="Screenshot 2023-06-16 090012" src="https://github.com/Prem-Kumar16/Dashboard_demo_2/assets/75419846/58580da4-1891-44c1-8f33-a968324f8a4e">

If you go to ec2 instances page, you will find two newly created instances named "Dashboard-demo-master" & "Dashboard-demo-worker". SSH into the instances using the key file  that you have previously downloaded

#### Important note, while SSH change the user name from root to ewaol i.e., instead of root@ip.eu-central-1.compute.amazonaws.com, you should use ewaol@ip.eu-central-1.compute.amazonaws.com

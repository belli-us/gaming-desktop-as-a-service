# AWS Gaming Desktop CloudFormation

This workspace contains a CloudFormation template at [cloudformation/g4dn-gaming-desktop.yaml](/Users/home/Documents/Code/DAAS/cloudformation/g4dn-gaming-desktop.yaml) that launches a single Windows gaming desktop on AWS with these defaults:

- `g4dn.xlarge`
- Windows Server 2022 from the public SSM AMI parameter
- `100` GiB encrypted `gp3` root storage
- Amazon DCV installed for low-latency remote desktop access
- Optional Elastic IP
- Optional inbound RDP
- SSM access and S3 read access for the AWS-hosted NVIDIA gaming driver bucket

## What It Deploys

The template creates one EC2 instance, one security group, one IAM role and instance profile, and optionally one Elastic IP. The instance is configured so that shutting Windows down stops the EC2 instance instead of terminating it.

## How To Deploy

1. Open CloudFormation in the AWS Region where `g4dn.xlarge` is available and where your account has sufficient G instance quota.
2. Create a new stack and upload [cloudformation/g4dn-gaming-desktop.yaml](/Users/home/Documents/Code/DAAS/cloudformation/g4dn-gaming-desktop.yaml).
3. Supply:
   - a `VpcId`
   - a public `SubnetId` with a route to an internet gateway
   - a strong `AdministratorPassword` using letters, numbers, and one of `! @ # % ^ * _ - + = : , . ?`
   - a restricted `AllowedClientCidr` if possible, ideally your public IP with `/32`
4. Leave `AssociateElasticIp=true` unless you already have a public IP strategy.
5. Wait for stack creation to finish. The template uses a CloudFormation signal so completion should indicate DCV finished installing.

## Connecting

After the stack completes, use the `DcvWebUrl` output or the `DcvNativeClientUrl` output. Log in with:

- Username: `Administrator`
- Password: the `AdministratorPassword` parameter you supplied

If you enabled RDP, you can also connect on TCP `3389`.

## NVIDIA Gaming Driver

Amazon DCV is installed automatically. The template also places helper scripts on the instance for the AWS NVIDIA gaming driver flow documented by AWS:

- `C:\Scripts\Download-NvidiaGamingDriver.ps1`
- `C:\Scripts\Register-NvidiaGamingLicense.ps1`

Recommended first-login flow:

1. Connect with DCV.
2. Open an elevated PowerShell session.
3. Run `C:\Scripts\Download-NvidiaGamingDriver.ps1`.
4. Run the downloaded NVIDIA installer that matches Windows Server 2022.
5. Reboot.
6. Run `C:\Scripts\Register-NvidiaGamingLicense.ps1`.
7. Reboot again.

## Important Caveats

- This is a single-user desktop, not a multi-user streaming platform.
- Many PC games work, but some anti-cheat systems and consumer Windows requirements can block play on Windows Server.
- `g4dn.xlarge` includes local NVMe instance storage, but this template keeps the requested persistent storage on the 100 GiB EBS root volume.
- Public IPv4 addresses incur AWS charges. Stopping the instance when not in use is important for cost control.

# AWS_VPN
𝗖𝗼𝗻𝗳𝗶𝗴𝘂𝗿𝗲 𝗮 𝗰𝗹𝗶𝗲𝗻𝘁 𝗩𝗣𝗡 𝗱𝗲𝗽𝗹𝗼𝘆𝗺𝗲𝗻𝘁 𝗮𝗹𝗹𝗼𝘄𝗶𝗻𝗴 𝘄𝗼𝗿𝗸𝘀𝘁𝗮𝘁𝗶𝗼𝗻𝘀 𝘁𝗼 𝗰𝗼𝗻𝗻𝗲𝗰𝘁 𝘁𝗼 𝗮 𝗔𝗪𝗦 𝗩𝗣𝗖

**Create a simple AD instance**:

<img width="1917" alt="STAGE1" src="https://github.com/MohammadSaadHaroon/AWS_VPN/assets/52069939/32deffe4-78a2-4255-9743-5665c07db3ea">


- Type `Directory Service` into the top search box and open the directory services console in a new tab.  
Or use directly open via https://console.aws.amazon.com/directoryservicev2/identity?region=us-east-1#!/directories  
- Click `Set up Directory`  
- Select `Simple AD` and click `Next`
- Choose `Small` (this is a demo, for larger deployments you might pick large, or choose alternative methods of authentication)
- Pick a `Directory DNS Name`, I'll use `corp.practice.org`(use your DNS in DNS name)  
- Pick a `Directory NetBIOS name`, I'll use `CORP`  
- Choose an `Administrator password`, it will need to be strong enough to meet the complexity requirements.  
- Enter the same password again in the `Confirm Password box  
- for `Directory description - Optional` put `Directory Service for AWS Client VPN Demo`  
- Click `Next`  
- Under VPC click the dropdown and pick `VPC2`(Use your desired VPC)  
- Under `Subnets` ideally pick `SB-PRIV-A` and `SB-PRIV-B` (but if one of those isn't available, pick 2 of the PRIV subnets)  
- Click `Next`
- Click `Create Directory

The directory will start provisioning, it will need to complete and move into the `Active` state before continuing to stage 2

`END OF STAGE1`

**AWS Client VPN - Certificates**

<img width="1917" alt="STAGE2" src="https://github.com/MohammadSaadHaroon/AWS_VPN/assets/52069939/ddfa5ef9-44f0-406f-bb61-f6c225954f42">

# Creating Certificates

# Server Certificate

This part of the demo involves downloading easy-rsa and using this to create certificates which will be imported into ACM. If using this in production, you can create server and client certificates for mutual authentication. To keep things simple (the focus is on ClientVPN itself) we will only be using the server certificate.

## Linux/macOS

- open a terminal
- cd /tmp 
- git clone https://github.com/OpenVPN/easy-rsa.git
- cd easy-rsa/easyrsa3
- ./easyrsa init-pki
- ./easyrsa build-ca nopass
- - practiceVPN
- ./easyrsa build-server-full server nopass
- aws acm import-certificate --certificate fileb://pki/issued/server.crt --private-key fileb://pki/private/server.key --certificate-chain fileb://pki/ca.crt --profile iamadmin-general

## Windows

- Open the OpenVPN Community Downloads page, download the Windows installer for your version of Windows, and run the installer. 
- Open the EasyRSA releases page and download the ZIP file for your version of Windows. Extract the ZIP file and copy the EasyRSA folder to the \Program Files\OpenVPN folder. 
- Open the command prompt as an Administrator, navigate to the \Program Files\OpenVPN\EasyRSA directory, and run the following command to open the EasyRSA 3 shell.
- EasyRSA-Start
- ./easyrsa init-pki
- ./easyrsa build-ca nopass
- ./easyrsa build-server-full server nopass
- ./easyrsa build-client-full client1.domain.tld nopass
- exit

- aws acm import-certificate --certificate fileb://pki/issued/server.crt --private-key fileb://pki/private/server.key --certificate-chain fileb://pki/ca.crt --profile iamadmin-general


- Type `ACM` or `Certificate Manager` into the search box at the top of the screen then right-click and open in a new tab.
- Verify that your certificate exists in the `us-east-1` region.  



`END OF STAGE2`

# AWS Client VPN - Create VPN Endpoint

<img width="1916" alt="STAGE3" src="https://github.com/MohammadSaadHaroon/AWS_VPN/assets/52069939/9ea0ab52-8710-422d-9157-d487b5d76c44">


# Create VPN Endpoint

- Type VPC in the services search box at the top of the screen, right-click and open in a new tab.  
- Under Virtual Private Network (VPN) on the menu on the left, locate and click `Create VPN Endpoint`  
- Click `Create Client VPN Endpoint`  
- For `Name Tag` enter `A4L Client VPN`  
- Under `Client IPv4 CIDR*` enter `192.168.12.0/22`  
- Click the `Server certificate ARN*` dropdown and select the server certificate you created in stage 2.  
- Under `Authentication Options` check `Use user-based authentication  
- Check `Active Directory authentication  
- Under `Directory ID*` choose the directory you created in Stage 1 (e.g. corp.animals4life.org) 
- Under `Connection Logging`, `Do you want to log the details on client connections?*` check `no`  
- For `DNS Server 1 IP address` and `DNS Server 2 IP address` we need to enter the IP addresses of the directory service instance. Go back to the tab with the directory service console open, click the directory service instance ID , locate the `DNS address` area and copy one IP into each of the DNS Server IP boxes.  
- Check `Enable split-tunnel  
- in the `VPC ID` dropdown select `A4L-VPC`  
- Ensure the `Default` SG is checked and the `A4L DefaultSG`
- Check `Enable self-service portal`  
- Click `Create Client VPN Endpoint`  
- Click `Close`  

At this point, the VPN endpoint is ready for configuration in the next stage

# AWS Client VPN - Configure VPN Endpoint & Associations

<img width="1916" alt="STAGE4" src="https://github.com/MohammadSaadHaroon/AWS_VPN/assets/52069939/90ad913d-1342-4eed-8538-26cdba9846e3">

**Please make sure the Directory Service created in the previous step is in an `Active` state.**  
**Please make sure you have created and imported the server certificate from stage 2 before starting stage3** 
**Please make sure the VPN endpoint has been created in the previous stage** 

# Associate ClientVPN Endpoint  

- From the `Client VPN Endpoints` area of the VPC console, select the `A4L Client VPN` endpoint.  
- Click the `Associations` tab and click `Associate`  
- Click the `VPC*` dropdown and click the `A4L-VPC`  
- Open in a new tab, the VPC, Subnets console https://console.aws.amazon.com/vpc/home?region=us-east-1#subnets:  
- Locate the subnet ID for the 3 private subnets in the A4L VPC  
- Click the `Choose a subnet to associate*` dropdown and pick the first available PRIV subnet from the list (PRIV-A, PRIV-B, PRIV-C)  
- Click `Associate`  then click `Close`  

The VPN Endpoint will now be associated, you need to pause here and wait for the state of the VPN endpoint to change from `Pending-associate` to `Available`


<img width="1917" alt="STAGE5" src="https://github.com/MohammadSaadHaroon/AWS_VPN/assets/52069939/30efcdd4-8efc-4f19-89b6-e68b8aa7df62">


# Download the ClientVPN Application & Config

- From the `Client VPN Endpoints` area of the VPC console, select the `A4L Client VPN` endpoint. 
- Click `Download Client Configuration`, Click `Download` and save the file.  
- Go to https://aws.amazon.com/vpn/client-vpn-download/ and download the client for your operating system
- Install the VPN Application
- Start the application
- Go to manage profiles
- add a profile
- load the profile you downloaded (client configuration) - use `A4L` for Displayname  

# Connect

- Connect to A4L VPN
- enter the `Administrator` username and password you chose in stage 1 for the Directory Service
- once connected open a terminal and `ping DIRECTORY_SERVICE_IP_ADDRESS' (you can get this from the DS Console)

Notice it doesn't work ? once more step, and thats authorization

# CLIENT VPN ENDPOINT

- Open the Client VPN Connection console https://console.aws.amazon.com/vpc/home?region=us-east-1#ClientVPNEndpoints:
- Select `A4L Client VPN`, Click `Associations`, Click `Disassociate`, Click `Yes, Dissasociate`
- Wait for the `Disassociate` to finish.
- Select `A4L Client VPN`, then click `Actions`, `Delete Client VPN Endpont`, Click `Yes, Delete`
- Wait for the Client VPN Endpoint to finish deleting

# CERT

- Type `ACM` or `Certificate Manager` into the search box at the top of the screen then right click and open in a new tab.
- Select the server certificate, Click `Actions`, Click `Delete`, Click `Delete`

# Directory Service

- Type `Directory Service` into the top searchbox and open the directory services console in a new tab.  
Or use directly open via https://console.aws.amazon.com/directoryservicev2/identity?region=us-east-1#!/directories  
- Select the directory created in stage 1
- Click `Actions` then `Delete directory`
- Type the name of the directory an then click `Delete`
- Wait for the directory to finish deleting before moving on.

# STACK

- Type `CloudFormation` into the top searchbox and open the CloudFormation console in a new tab. https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringStatus=active&filteringText=&viewNested=true&hideStacks=false  
- Select the `A4L` Stack, Click `Delete`, click `Delete Stack`  

# LOCAL 

- Open Manage profiles
- Delete the A4L profile
- optionally delete the AWS Client VPN Software

  #Reference
  https://github.com/acantril/learn-cantrill-io-labs/tree/master/aws-client-vpn

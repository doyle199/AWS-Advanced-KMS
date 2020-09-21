# AWS-Using-KMS
AWS Using KMS

You must have an AWS account, a user with the correct permissions to generate polices, create/modify roles in IAM, run CloudFormation templates, and launch EC2 instances, and a VPC (each region creates a default VPC for you now).

Download this template to your workstation: https://github.com/doyle199/AWS-Using-KMS/blob/master/KMS_Template_1.jscsrc. Navigate to the AWS CloudFormation console and click create stack.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Stack.png)

On the specify template page, select template is ready and upload a template file. Choose the template that was just downloaded to one’s workstation and click next.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Stack%20_1.png)

On the specify stack details page, create a stack name and click next.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Name_Stack_1.png)

On the configure stack details page leave the defaults and click next. On the review page check the acknowledgment box and click create stack.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Stack_Options.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Stack_Complete.png)

Navigate to the EC2 console and click on instances in the left menu then launch instance.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Launch_Instance.png)

For step one, select a Linux AMI.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Linux_AMI.png)

For step two, choose an instance size and click next: configure instance details.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Instance_Type.png)

For step three, choose a VPC with public access. Make sure auto assign public IP is set to enabled. For the IAM role select the KMSWorkshopInstanceInitRole that was created by the last CloudFormation stack then click next.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Configure_Instance_Details.png)

Click next for step four and five. Allow SSH from one’s workstation IP and click review and launch.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Configure_SG.png)

On the review page select launch. Choose a keypair or make a new one. Check the acknowledgement box. Click on launch instances. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Select_Key.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/instance_run.png)

The role can also be attached by clicking on the instance then the actions dropdown. From there click on instance settings then attach/replace IAM role. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/instance_actions.png)

On the next page select the role and click apply.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/select_role.png)

Test that it’s working by clicking on the instance and selecting the connect button near the top and connect to it via a terminal. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Connect_1.png)

Update the instance with the command"sudo yum update"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/CLI_connect.png)

To create customer managed keys, run the command "aws configure". One may need the correct IAM Access Key and Secret Access Key. type the correct region code.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/customer_managed_1.png)

Run the following command "aws kms create-key"

The key was created. If the key does not create one must create a policy to attach to the KMSWorkshop-InstanceInitRole.
Navigate to the IAM console, click on roles in the left menu. Search for KMSWorkshop-InstanceInitRole and click on it.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Role_1.png)

Click on attach policies.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/attach_policies.png)

Search for AWSKeyManagementSystemPowerUser and click on the dropdown and then the JSON button. Scroll down to the allow section to see what this allows to the role to do. Check the box next to the role and click attach policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Power_User.png)

Take note of the Key ID after running the command "aws kms create-key"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create-key.png)

To create a key alias named “FirstCMK” enter the following command into the terminal using one’s Key ID. "aws kms create-alias --alias-name alias/FirstCMK --target-key-id 'your-key-id'"

Next one will generate a CMK by importing one’s own key material. Type the following command into the terminal and take note of the Key ID "Aws kms create-key –origin EXTERNAL"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/external.png)

To download the public key and import token to wrap the key, run the following command using the last Key ID "aws kms get-parameters-for-import --key-id external-key-id  --wrapping-algorithm RSAES_OAEP_SHA_1 --wrapping-key-spec RSA_2048"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/wrap.png)

Type “nano” into the terminal. Past the contents of the PublicKey into the editor, exit and save the file as pkey.b64. Do the same for the ImportToken contents and save it as token.b64. run the command "ls -l" to see the files. •	Next, run the following two commands: "openssl enc -d -base64 -A -in pkey.b64 -out pkey.bin" "openssl enc -d -base64 -A -in token.b64 -out token.bin"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/nano.png)

Now one will create the import material and encrypt it for the import using the OpenSSL library. Run the following command to create a key stored in a genkey.bin file: "openssl rand -out genkey.bin 32"

Run the following command to wrap the key with the pkey.bin file one created: "openssl rsautl -encrypt -in genkey.bin -oaep -inkey pkey.bin -keyform DER -pubin -out WrappedKeyMaterial.bin" Which saves it in a file called WrappedKeyMaterial.bin.

To import the key material run the following command using ones KeyID: "aws kms import-key-material --key-id your-key-id --encrypted-key-material fileb://WrappedKeyMaterial.bin --import-token fileb://token.bin --expiration-model KEY_MATERIAL_EXPIRES --valid-to 2021-02-01T12:00:00-08:00"

If the operation failed, it’s because the role needs more access. Navigate to the IAM console, click on policies in the left menu, then click create policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Policy.png)

Type KMS into the search then click on it. Click the dropdown for Actions and Write under access level and check the box next to ImportKeyMaterial.






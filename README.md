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

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/importKey.png)

Next click the dropdown for all resources. Click all resources then review policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/review_policy.png)

Name the policy KMS-Workshop-ImportMaterialPermissions then click create policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/name_policy.png)

Click on roles, search for KMSWorkshop-InstanceInitRole and click on it. Click attach policies. Search for KMS-Workshop-ImportMaterialPermissions, check the box next to it and click attach policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/attach_permissions.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/import_2.png)

Give the new key an alias called “ImportedCMK” by running the following command using one’s KeyID "aws kms create-alias --alias-name alias/ImportedCMK --target-key-id 'YOUR-ARN'"

Run the following command to see the key alias in the CLI: "aws kms list-aliases"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/aliases.png)

To rotate the AWS KMS CMKs, enable AWS automatic key rotation by running the following command using the KeyID for the FirstCMK key "aws kms enable-key-rotation --key-id YOUR-KEY"

To rotate CMKs generated with AWS key material run the following command "aws kms create-key"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create-key_2.png)

Run this command to update the FirstCMK with this new key using the new KeyID "aws kms update-alias --alias alias/FirstCMK --target-key-id KeyId". If the role does not have permission to update permissions navigate to IAM console and create a new policy. Choose KMS for service. Check the boxes shown in the screenshot below. Select all resources and click review policy. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/KMS_permissions.png)

Name it KMSWorkshop-RotationDisableOps and click create policy. Attach the policy to the role as before. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Attach_Policy_2.png)

The FirstCMK alias is now associated with the new KeyID

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/new_KeyID.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create_policy_3.png)

To rotate keys generated with one’s own key material, automatic is not an option so one must update the alias pointer. Create one’s own key as before, going through all the steps.

Enter the following code to point the ImportedCMK to the ImportedCMK2 KeyID "aws kms update-alias --alias alias/ImportedCMK --target-key-id KeyID"

To disable a key run the following with the correct KeyID "aws kms disable-key --key-id your-key-id"

To enable it again run the following command with the correct Key ID "aws kms enable-key --key-id your-key-id"

To schedule deletion of AWS created keys, a minimum of 7 days and a maximum of 30 days run the following with the correct KeyID or ARN "aws kms schedule-key-deletion --key-id your-key-id --pending-window-in-days 7"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/schedule_deletion.png)

To delete ones own keys they can be scheduled or deleted on demand run the following command "aws kms delete-imported-key-material --key-id  your-key-id"

To use envelope encryption, first one must create the secret text to encrypt using the following command "sudo echo "Sample Secret Text to Encrypt" > samplesecret.txt"

Next one must generate the data key referencing the ImportCMK one created using the following command "aws kms generate-data-key --key-id alias/ImportedCMK --key-spec AES_256 --encryption-context project=workshop"

If that doesn’t work navigate to the IAM dashboard, select policy then create policy. Choose the KMS service and in the write dropdown under actions select GenerateDataKey, GenerateDataKeyWithoutPlaintext, Encrypt, Decrypt, and under tagging, TagResource, and UntagResource. Then choose all resources and give the policy the name KMSWorkshop-AdditionalPermissions and create policy. Then attach it to te KMSWorkshop-InstanceInitRole. Then it will work.

Take note of the Plaintext, KeyId, and CiphertextBlob information.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/generate_key.png)

To encrypt the secret text using OpenSSL library, un the following command using the plaintext key "echo 'KEY-HERE' | base64 --decode > datakeyPlainText.txt

Run the following command using the CipherTextBlob key which encrypts the samplesecret.txt with the datakeyplaintext.txt and saves it to encryptedsecret.txt "echo 'CIPHER-TEXT-BLOB-KEY-HERE' | base64 --decode > datakeyEncrypted.txt"

Run the following code to encrypt the file "openssl enc -e -aes256 -in samplesecret.txt -out encryptedSecret.txt -k fileb://datakeyPlainText.txt"

Run the following code to see if it was encrypted "more encryptedSecret.txt"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/encrypted.png)

Next enter the following command to delete the plaintext key "rm datakeyPlainText.txt"

To decrypt the encrypted secret text, run the following command "aws kms decrypt --ciphertext-blob fileb://datakeyEncrypted.txt"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/failed_1.png)

It fails because one needs to provide the encryption context to decrypt. Run the following command "aws kms  decrypt --encryption-context project=workshop --ciphertext-blob fileb://datakeyEncrypted.txt"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/unencrypted.png)

This gets the Plaintext key back with AWS KMS and the appropriate CMK. Next one must decode it from base64 and use it to decrypt the encrypted secret file encryptedSecret.txt. Run the following command using the plaintext key "echo 'KEY-HERE' | base64 --decode > datakeyPlainText.txt"

Next run the following code to decrypt "openssl enc -d -aes256 -in encryptedSecret.txt -k fileb://datakeyPlainText.txt"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/to_encrypt.png)

It has encrypted.

To use serverside envelope encryption, navigate to the EC2 console. Click on volumes in the left menu then create volume.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Volume.png)

Make sure the AZ is the same as the instance because the disk can only be attached to instances in the same AZ. Check the encryption checkbox then select ImportedCMK for the master key.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create_volume_KMS.png)

Create a tag Name: WorkshopEBS. Click on Create volume.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Volume_2.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/volume_success.png)

Now a volume is encrypted that can be attached to an instance. Select the volume that was just created. Click the actions dropdown and select attach.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/attach_volume_2.png)

Choose the correct instance and click attach.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Attach_volume_4.png)

Run the following command in CLI to see the disk "lsblk"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/lsblk.png)

Encrypted data can now be stored. AWS Encryption SDK can also be used to create client-side encryption.

Encrypting using AWS KMS with no data key. Encrypt using CLI and decrypt using CMK.

Create a new secret file by running the following command "echo "New secret text" > NewSecretFile.txt"

Encrypt it with CMK and use an encryption context that will be needed later by running the followign command "aws kms encrypt --key-id alias/ImportedCMK --plaintext fileb://NewSecretFile.txt --encryption-context project=kmsworkshop --output text  --query CiphertextBlob | base64 --decode > NewSecretsEncryptedFile.txt"

The new output file is NewSecretsEncryptedFile.txt. Check it with the following command "cat NewSecretsEncryptedFile.txt"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Encrypted_2.png)

It’s encrypted. To decrypt it run the following command "aws kms decrypt --ciphertext-blob fileb://NewSecretsEncryptedFile.txt --encryption-context project=kmsworkshop --output text --query Plaintext | base64 --decode > NewSecretsDecryptedFile.txt"

To work with a WebApp, In the terminal make sure one is in the home directory with the following command "Pwd"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/pwd.png)

Make a new directory and install boto3 python library with the following two commands: "sudo mkdir SampleWebApp" "sudo pip install boto3"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/boto_3.png)

In the directory download the sample WebApp with wget with the following two commands: "cd SampleWebApp" "sudo wget  https://raw.githubusercontent.com/aws-samples/aws-kms-workshop/master/WebApp.py"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/samplewebapp.png)

A python application called WebApp.py was downloaded. Get the instance public IP on the EC2 console and save it for later.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Instane_ID.png)

Run the following command to run the webserver "sudo python WebApp.py 80"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/webServer.png)

Navigate to the WebApp using the public IP in a browser. If it doesn’t work check the instance security group for HTTP(S) traffic. In the instance description tab find the security group. Click on it then click on it again on the next page. Click edit inbound security rules. Click add rule and add HTTP(S) from anywhere and click save rules.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/SG_1.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/SG_2.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/SG_3.png)

Now check the public IP in a browser.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/WebApp%20Uploader_1.png)

Create a sample text file using a text editor and save the file as SampleFile-KMS.txt. Upload it using the WebApp.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/File_upload.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Upload_Success.png)

The file now appears in the S3 - Bucket: kms-workshop. One can click on it to display it.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/WebApp_Upload_2.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/sample_file.png)

To add encryption to the WebApp using server-side encryption with AWS KMS and the CMK that was created before, stop the server from running by pressing Control+C twice in the terminal. Download the version of the WebApp that adds server-side encryption with the following command "sudo wget https://raw.githubusercontent.com/aws-samples/aws-kms-workshop/master/WebAppEncSSE.py"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/serverside%20encryption.png)

Use the following command to list find ones CMK KeyID "aws kms list-aliases" and take note of the KeyID.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/KeyID.png)

Run the following command to start the server with encryption and enter the KeyId when it asks "sudo python WebAppEncSSE.py 80"

Now go back to the browser and refresh the WebApp page. Upload the SampleFile-KMS.txt file again.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/now_encrypted.png)

Navigate to the S3 console. Click on the kmsworkshop bucket. Click on the SampleFile-KMS.txt.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/S3-1.png)

It says that it has server-side encryption and shows the KMS Key ID

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/KMS_Key_3.png?raw=true)

To check the policy for an AWS KMS run the following command with the correct KeyID "aws kms list-key-policies --key-id your-key-id"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/default.png)

Currently the policy is default. To see what it contains run the following command with the correct Key ID "aws kms get-key-policy --key-id your-key-id --policy-name default"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/default_contents.png)

The default policy gives the AWS root user account full access to the CMK. Next one will modify the permission of the role assigned to the instance allowing it to encrypt but not decrypt. Navigate to IAM, click on roles in the left, search for KMSWorkshop-InstanceInitRole and click on it. Find and click on the attached policy KMSWorkshop-AditionalPermissions and click on Edit Policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/KMS_Permissions_2.png)

Click on the JSON tab and remove the kms:Decrypt action. Click review policy and then save policy.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/decrypt_4.png)

Now run the server again with the following command and Key ID "sudo python WebAppEncSSE.py 80". Upload a new file to S3 with the WebApp and then try to download it. It will fail.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Fail_3.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/fail_4.png)

To enforce least privilege access and ensure the encryption role is only used form one’s account and not subject to cross-account role access policies one shall use a handy key policy. Navigate to the IAM console, click on roles, search for KMSWorkshop-InstanceInitRole and click on it. Find the Role ARN near the top and save it.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/role_3.png)

Navigate to the KMS dashboard, Click on customer managed keys in the left menu. Find the ImportedCMK keys.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/KeyID_5.png)

Scroll down to key policy, click edit and replace the script with the following then click save changes https://github.com/doyle199/AWS-Using-KMS/blob/master/template1.jscsrc. Use the correct user account ID number in the 3 places it says your-account-id

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Edit_Key_Policy.png)

Now the key can only be used with that account. Enforcing least privilege can be used by ensuring that CMK can only be called by one’s role. Enter the following in the key policy like before using the account ID number: https://github.com/doyle199/AWS-Using-KMS/blob/master/template2.jscsrc

One can also include conditions with a key policy. To add MFA replace the policy with the following script as before. https://github.com/doyle199/AWS-Using-KMS/blob/master/template3.jscsrc

The following command would enforece communication through an endpoint "aws kms list-keys --endpoint-url https://vpce-xxxxxxxxa-xxxxxx.kms.your-region.vpce.amazonaws.com"

one can use the key policy to only allow certain operations from the VPC endpoint. The following policy will make AWS KMS only available internally within the AWS network for certain sensitive operations. https://github.com/doyle199/AWS-Using-KMS/blob/master/template4.jscsrc

Tagging can be done during key creation or with a command like "aws kms tag-resource --key-id your-key-id --tags TagKey=project,TagValue=kmsworkshop"

To list the resource tags use the following command "aws kms list-resource-tags --key-id your-key-id"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/TagKey.png)

One can use monitoring and logging with AWS KMS using CloudTrail. To see CloudTrail logs, create a data key with the corresponding AWS KMS the following command "aws kms generate-data-key --key-id alias/ImportedCMK --key-spec AES_256 --encryption-context project=workshop"

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/trials.png)

That action has been logged in CloudTrail. Navigate to the CloudTrail console and select event history in the right menu. For the filter dropdown select event name and enter GenerateDataKey in the next box.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Event.png)

The GenerateDataKey operations are shown.	Click on an event for more information.	There are many more ways to filter the information.

To make real time AWS KMS notifications with AWS CloudTrail, Amazon CloudWatch, and Amazon SNS, navigate to the SNS console. Click topics in the left menu and then create topic.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create_topic.png)

Enter a name and display name and click create.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/success.png)

Take note of the ARN. Click subscriptions in the left menu and then create subscription. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create_subscriptions.png)

Enter the topic ARN. Choose email for the protocol and enter the desired email, click create subscription.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/success3.png)

Confirm the subscription in one’s email.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/subcription_email.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/confirmed.png)

Navigate to the Amazon CloudWatch. Click on events in the left menu and then Get Started. 

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/CloudWatch%20Events.png)

Leave event pattern checked, select events by service in the dropdown. Select CloudTrail for the service name and AWS API call via CloudTrail for the event type. Select the circle for specific operations and enter GenerateDataKey in the box. On the right side of the screen select the +Add target button, on the first row change it to SNS topic. Then select snsworkshop for the topic and click configure details.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create_rule_2.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_rule_3.png)

If CloudTrail is not enabled on one’s account, there will be a pop up. Go to CloudTrail and select create trail.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/warning.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/not_enabled.png)

Create a trail name and enter the name of one’s S3 bucket, create one if needed, and select create.

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/Create_Trail.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/trail_bucket.png)

![alt text](https://github.com/doyle199/AWS-Using-KMS/blob/master/create_trail_2.png)

Continuing the CloudWatch rule creation on step 2 configure rule details enter a name and click create rule.












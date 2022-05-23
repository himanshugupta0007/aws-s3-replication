## What is AWS S3 Same Region Replication (SRR)?

AWS S3 Same Region Replication allows replicating the newly uploaded objects to S3 destination buckets asynchronously and automatically present in the same region as the source bucket. Still, it could be in a different Availability Zone.

Users can configure the replication rule so the rule can identify the objects to replicate using prefix, tag, or bucket through AWS CLI, Management Console, and AWS SDK.

Also, users can configure to replicate S3 objects across cross-account in a single region or across cross-region. 

S3 Cross-account and Cross-Region replication we will cover in upcoming tutorials

![AWS S3 - Same Region Replication .png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653307535741/YbgScthj5.png align="left")

## Use Cases of AWS Same Region Replication (SRR)

- When the user wants to ensure a replica of their objects for data compliance requirements
- The user wants to update the ownership of objects to protect against accidental data deletion.
- You can configure the in-Region replication to aggregate the logs from multiple S3 buckets to a single S3 bucket for more explicit processing.
- When the user wants to change their objects' storage class, you can use the S3 object lifecycle.
- The user wants to configure live data replication between multiple accounts, such as Stage and Production accounts.

## What are the prerequisites for AWS to replicate your objects?

- User must enable versioning at S3 source and destination bucket
- Proper IAM policies and bucket policies must be created to give AWS S3 Permission to replicate the objects on the user's behalf
- If the source bucket has object lock enabled, the destination bucket also must have enabled the same

## What is replicated with the AWS replication configuration rule?

- The objects uploaded after you create the replication configuration are replicated automatically. In addition, you can use the AWS S3 Replication Batch job to replicate existing objects.
- Object's metadata from the source object to the replicas
- Only objects in the source bucket for which the bucket owner has permissions to read objects and access control lists (ACLs)
- Object tags and S3 Object Lock retention information, if any

## What is not replicated?

- Already replicated objects present in the bucket have the status "REPLICA."
- Objects in the source bucket have already been replicated to a different destination. So, for example, if you change the destination bucket in an existing replication configuration, Amazon S3 won't replicate the objects again.
- Objects in the source bucket that the bucket owner doesn't have sufficient permissions to replicate Actions performed by the bucket's lifecycle configuration

## Steps to configure the AWS Replication Rule - Same Region

### a. Create a Source and Destination Buckets

- Create an S3 source bucket and an S3 destination bucket in your AWS Management Console in the same AWS Region.
- Make sure your bucket's name is unique and DNS compatible; you must enable bucket versioning while creating buckets.

![AWS_S3_Replication_Configuration_Source_Destination_bucket.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653302355231/8yV6d_akD.png align="left")

### b. Set up an AWS Identity and Access Management (IAM) role and policy

- User must create a role using which Amazon S3 can assume to replicate objects on the user's behalf
- Go to IAM Console and select Roles from the left navigation under Access Management.
- Choose to create a role from Role Dashboard, select AWS Service as a trusted Entity, and choose S3 from the bottom dropdown for Use Case and press Next.

![AWS_S3_Replication_Rule_Create_Role_Dashboard.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653302497078/ZgZw2eool.png align="left")

- At Permission, choose Create Policy and select JSON tab, and copy-paste below policy code.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetReplicationConfiguration",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::<source-bucket>"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObjectVersionForReplication",
                "s3:GetObjectVersionAcl",
                "s3:GetObjectVersionTagging"
            ],
            "Resource": [
                "arn:aws:s3:::<source-bucket>/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ReplicateObject",
                "s3:ReplicateDelete",
                "s3:ReplicateTags"
            ],
            "Resource": "arn:aws:s3:::<destination-bucket>/*"
        }
    ]
}
``` 
- Replace the source bucket and destination bucket name according to your respective name
- The access policy grants the below permissions
<!-- wp:list -->
<ul><li>
- **s3:GetReplicationConfiguration** and **s3:ListBucket**: It allows S3 to get information on replication configuration and buckets on the source bucket</li>
<li>
- **s3:GetObjectVersionForReplication** and **s3:GetObjectVersionAc**l: This Permission allows Amazon S3 to get a specific object version and access control list (ACL) associated with the objects.
</li>
<li>
- **s3:ReplicateObject** and **s3:ReplicateDelete**: This Permission allows Amazon S3 to replicate objects or delete markers to the destination bucket
</li><ul>
<!-- /wp:list -->

- Click on Tags, add tags if you want to, and Press Next.
- Name the policy and click on Create policy.

![AWS_S3_Replication_Rule_Create_Policy_permission.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653304404225/32FhBk5Sw.jpeg align="left")

- On Add Permission page, refresh at Permission policies. You can see your new policy created listed. Select your policy and press Next.

![AWS_S3_replication_Create_role_permission.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653304721332/12t-f2b_x.jpeg align="left")

- Name your role and description, and press Create Role.

### c. Create Replication Configuration Rule

- Open the S3 console, https://console.aws.amazon.com/s3/
- Select your source bucket, choose the Management tab, and scroll down to Replication Rules. Click on Create Replication Rule

![AWS_S3_Source_Bucket_Replication_Rule_Tab.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653304839022/FeSM5Osof.jpeg align="left")

<!-- wp:list -->
<ul>
<li>
Provide Replication Rule Name

![AWS_REplication_Rule_Create_Replication_Rule_Configuration.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653304801418/58hej89Dr.png align="left")
</li>
<li>
Under Source Bucket 
<ul>
<li>You can provide the prefix of objects you want to replicate</li>
<li>We will select "Apply to all objects in the bucket" for this tutorial.</li>

![AWS_S3_Replication_Rule_Source_bucket_Rule_scope.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653304914701/t9QYeBmiz.jpeg align="left")

</ul>
</li>
<li>
 Under Destination Bucket
<ul>
<li>Select "Choose a bucket in this account" and press browse S3 to get the bucket list.</li>
<li>Select your destination bucket and click on Choose Path.</li>


![AWS_S3_Replication_ule_Choose_DestinationBucket.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305057420/4WdHBtcez.jpeg align="left")

</ul>
</li>
<li>
 Under IAM Role
<ul>
<li>Select "Choose from existing IAM roles" and select role created earlier.</li>


![AWS_S3_Replication_Rule_Choose_IAM_Role.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305086869/HNq1zh2HI.jpeg align="left")

</ul>
</li>
<li>
Under Destination Storage Class, you have an option to change the storage class of replicating objects. For this tutorial, let's keep the storage class the same as the source bucket.
</li>
<li>
Additional replication options
<ul>
<li>**Replication Time Control (RTC) **- S3 Replication Time Control (S3 RTC) helps you meet compliance or business requirements for data replication and provides visibility into Amazon S3 replication times. S3 RTC replicates most objects you upload to Amazon S3 in seconds and 99.99 percent of those objects within 15 minutes..</li>
<li>
**Delete marker replication** - AWS S3 allows you to replicate the delete marker to your destination buckets. Amazon S3 behaves as if the object was deleted in both source and destination buckets.
</li>
<li>
**Replica modification sync** - Amazon S3 replica modification sync can help you keep object metadata such as tags, ACLs, and Object Lock settings replicated between replicas and source objects. When replica modification sync is enabled, Amazon S3 replicates metadata changes made to the replica copies back to the source object, making the replication bidirectional.
</li>
</ul>


![AWS_S3_Replication_Rule_Additional_Configuration.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305123303/xP8U1dPLr.jpeg align="left")

</li>
</ul>
<!-- /wp:list -->

- Press Save to create your replication rule
- AWS prompts if you want to replicate the existing objects in the bucket. For now, please select No and press to submit. We will cover the replication of existing objects in another tutorial.

![AWS_S3_Replication_Rule_Ignore_Existing_Ojects.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305183432/RNwKYeCze.jpeg align="left")

- Once you create the replication rule, your configured rule will be visible under the Replication Rule with the status Enabled.

![AWS_S3_Replication_Rule_Status.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305224443/SSKIYciBu.jpeg align="left")

### d. Test the S3 bucket Replication

- Now, it's time to test the replication. First, move to your source bucket and upload any file using the upload option.

![AWS_S3_Replication_Rule_Upload_Objects.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305272106/q8lNKb94_.jpeg align="left")

- Wait for a couple of minutes and check your destination bucket, where the Object from the source bucket is replicated and present in the destination bucket.

![AWS_S3_Replication_Rule_Destionation_bucket.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305320650/ErLNi1c2C.jpeg align="left")

- You can also check the status of Object replication at the console.
<!-- wp:list -->
<ul>
<li>Go source bucket and select the Object you uploaded</li>
<li>Under the Object management overview, you can see the status of Replication as **COMPLETED**</li>

![AWS_S3_Replication_Rule_Source_bucket_replication_status.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305430196/ITrF4UT-j.jpeg align="left")

<li>Follow the same Step at the destination bucket, where you can see the Replication status as **REPLICA**</li>

![AWS_S3_Replication_Rule_Replicated_Object_Status.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653305557422/BBkDYDT14.png align="left")

</ul>
<!-- /wp:list -->


## Conclusion

This article teaches you how to set up AWS S3 Same Region Replication easily and answers all your queries regarding it. Furthermore, it provides a brief introduction of various concepts related to it & helps the users understand them better and use them to perform data replication.

The related documents and files are present on [Github URL](https://github.com/himanshugupta0007/aws-s3-replication-files)

Feel free to provide your feedback and subscribe to our newsletter for upcoming posts.

Resources used:

[AWS S3 Replication Documents](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication-example-walkthroughs.html)

To learn about AWS Lambda:

[Debug Lambda Locally](https://medium.com/programmers-block/how-to-debug-aws-lambda-java-function-in-eclipse-locally-a2bd172b423d)









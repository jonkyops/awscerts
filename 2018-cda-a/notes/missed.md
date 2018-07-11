# Missed Questions

- regions > edge locations - false
  - didn't know, there are edge locations in every major city
- how many regions - 16
  - didn't know, 42 AZs too
- What does an AWS Region consist of? - An independent collection of AWS computing resources in a defined geography.
  - chose -                             A distinct location within a geographic area designed to provide high availability to a specific geography. was a toss up

## IAM

- The AWS sign-in endpoint for SAML is https://signin.aws.amazon.com/saml. - True
  - Didn't know

## S3

- What is the largest size file you can transfer to S3 using a PUT operation? - 5GB
  - thought it was 5tb from largest size that can be stored
- You are using S3 in AP-Northeast to host a static website in a bucket called "acloudguru". What would the new URL endpoint be?
  - Got it correct, but remember that there is NO www in the address. correct url for static website hosting: http://bucketname.s3-website-region.amazonaws.com

## dynamodb

- provisionedthroughputexceeded (just know this one)
- What is the API call to retrieve multiple items from a DynamoDB table? batchgetitem
  - correct, but remember that it's NOT plural

## Other services

- You are designing a new application that processes payments and delivers promotional emails to customers. You need 
  to ensure that the payment process takes priority over the creation and delivery of emails. How might you use SQS to achieve this.
  - use two queues and process the payments one before checking the other one
  - wasnt discussed in lesson
- Your EC2 instances download jobs from an SQS queue. However, they are taking too long to process the messages. What API call can you use to extend the length of time to process the jobs?
  - ChangeMessageVisibility
  - wasnt discussed in lesson
- max sqs message size
  - 256kb
- SNS messages cannot be customized by protocol type.
  - not discuessed in lesson
- elastic beanstalk is not free and you must pay for the resources
  - answered true, was false. need to double check this one, not sure if test was right
- What is the maximum visibility of an SQS message in a queue?
  - piss poor wording, meant invisibility timeout. answered 14 days, was 12 hours
- Max longpolling timeout - 20 sec
  - not discussed

# dev theory quiz
- When deploying application code to EC2, the AppSpec file can be written in which language?
  - answered json or yaml, was yaml
- When deploying application code to Lambda, the AppSpec file can be written in which language? (select 2) 
  - got this right, but need to remember, since lambda/ec2 are different
- You want to recieve an email whenever a user pushes code to your CodeCommit repository, how can you configure this?
  - was not discussed, but still got right
  - Configure Notifications in the console, this will create a CloudWatch Events rule to send a notification to an SNS topic which will trigger an email to be sent to the user
    - main thing is that there is an option for 'notifications' in the codecommit service that will do thsi for you

# iam
<!-- TODO: got this right and understood, but review -->
- Amazon Cognito provides Web Identity Federation with which of the following features? (Select 2) (Choose 2)
  - Single sign-on for Active Directory users
  - Multi-Factor Authentication
  - * Synchronisation of user data across multiple device types
  - * Sign-up and sign-in to your applications
- What does Cognito use to create unique identities for users and authenticate them with Web ID providers?
  - chose user pools, was identity pools

# IAM Federation

## Process

More detailed writeup here, but agree with fsr's answer, those documents contain lots of helpful details!

1.  Employee enters their username and password into the application, in this case an ERP app.
2.  ERP application passes the username and password via API call to Identity Broker
    The application makes an API call to the Identity Broker server (for example via encrypted HTTPS with username and password in the body of the POST request). The Identity Broker server is an EC2 web server (or servers, or possibly API Gateway coupled with a Lambda script, etc.) that you have deployed an app to which allows it handle steps 2 - 5.
3.  Identity Broker authenticates the username and password with the organization's LDAP directory (e.g. Active Directory), to validate the employee's identity.
    This means that the Identity Broker communicates to the org's AD servers to validate that the username and password are valid. AD replies back yea or nay. The assumption is that AD replies back that the credentials are valid, so that we can proceed to step 4.
4.  Identity Broker calls the GetFederationToken function using IAM credentials. The call must include an IAM policy and a duration (1 to 36 hours), along with a policy that specifies the permissions to be granted to the temporary security credentials.
    Good reference document for this step (which is the same one fsr referenced in their answer):
    https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html
    This Identity Broker makes a programmatic call to the AWS STS via the GetFederationToken API call. Per reference document this programmatic call must be made using the secret key associated with a permanent IAM user (i.e. one you have created in IAM, not one that you have federated from elsewhere). The programmatic call GetFederationToken requires several parameters:
    DurationSeconds (900 seconds - 129600 seconds, the duration you wish the temporary credentials to be valid for)
    Name (the username or some identifying label for the user you are asking STS to issue temporary credentials to)
    Policy (JSON formatted IAM policy text with permissions you want to be granted to the temporary credentials)
5.  STS confirms that the policy assigned the IAM user making the programmatic call to GetFederationToken (in step 4) has permission to create new tokens. If it does, it then returns four values back to the Identity Broker:
    Credentials (contains three values)
    SessionToken - token text that can be used like a certificate to validate that IAM generated the following two credentials
    AccessKeyId - text that can be used in place of a username for programmatic calls using the temporary credentials
    SecretAccessKey - text that can be used in place of a password for programmatic calls using the temporary credentials
    FederatedUser (temporary username / identifier for the federated user based on the "Name" parameter passed to STS in the call in step 4)
    PackedPolicySize (percentage value indicating the size of the JSON formatted policy parameter passed to STS in the call in step 4. This is returned back to the IdentityBroker as a validation check that the JSON formatted policy size was not too large in size to be used and was not empty or invalid)
6.  The Identity Broker returns the temporary security credentials to the original ERP application that the user was attempting to log into.
    This means that the Identity Broker will reply back to the original API call in step 2 with the temporary credentials (e.g. with a 200 OK with the credentials in the body of the 200 OK response) granting it access to AWS resources as if it were an IAM user in AWS.
    7-9. In these steps, the user is logged into the ERP app and performing application functions that require writing changes to a document in S3. It doesn't matter what those changes are, could be just about anything ERP-related requiring document storage, but the main thing here is that the ERP app is going to attempt to make those changes to S3 on behalf of the user while using the temporary credentials that STS granted to the user.
7.  The data storage application uses the temporary security credentials (including the token) to make requests to Amazon S3.
    This means that the ERP app makes a programmatic call to S3 to read or write something (e.g. a POST call to write a new object into an S3 bucket). When making the programmatic call, the ERP app pretends to be the user by using the user's temporary AccessKeyId or SecretAccessKey. However, as an extra security check, the ERP app must also pass the SessionToken text along with those credentials, just to say "Not only am I providing these temporary credentials, here is the token text certifying that you issued those credentials to this temporary user, which you can verify." Think of this kind of like swiping your debit card at a store and entering a PIN, but then the store employee asks to see a photo ID as well, just to be sure it's really you.)
8.  S3 uses IAM to verify that the credentials allow the requested operation on the given S3 bucket and key.
    Back in step 4, the Identity Broker call to STS included a policy text in JSON format. That policy text had to contain permissions to do stuff with S3, if your app is going to need to do stuff with S3. This step is just S3 validating via IAM that the temporary credentials it is receiving from the ERP app have permission to actually do that stuff with S3.
9.  IAM provides S3 with the go-ahead to perform the requested operation.
    If IAM validates that the policy for those temporary credentials have permission to do stuff with S3, it replies back to S3 that it's OK to proceed. S3 then writes the object that the ERP app wanted to write on behalf of the user.

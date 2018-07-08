# Dev

## IAM

- Policies - Can be assigned to groups, roles, or users. defines permissions
- Security Token Service (STS)
  - Federation (typically active directory)
  - Uses Security Assertion Markup Language SAML
  - Temp access based off the users AD creds. Does not need to be a user in IAM
  - SSO allows signon w/o iam creds
- Federation w/ mobile apps
  - use fb/amz/goog or other openid providers to log in
- cross account access
  - lets users from one aws account access resources in another
- Terms:
  - Federation - combining or joining a list of users in one domain (like iam) with a list of users in another domain (like AD or fb)
  - Identity Broker - service that allows you to take an identity from point a and join it (federate it) to point b
    - doesn't come out of the box, need to make one yourself
  - identity store - services like ad, fb, goog, etc
  - identities - user of a service like fb, etc

### STS

- Example setup 1:
  - employees get on vpn, log into company app
  - company app passes credentials to identity broker
  - broker verifies against AD
  - broker asks for token from STS (using GetFederationToken function)
    - call must include iam policy and duration(1-36 housrs) and policy that specifies permissions to be granted to temp security credentials
  - broker gives token back to app
  - app accesses s3
  - In the exam
    - Develop (in house) an Identity Broker to comm with ldap and sts
    - broker always authenticates with ldap first, THEN with STS
    - app then gets temp access to aws resources
- Examples setup 2:
  - develop broker to communicate with ldap and sts
  - broker always auths with ldap first, gets an iam role associated with a user
  - app then authenticates w/ sts and assumes the role
  - app uses the role to interact w/ s3
  - in the exam
    - dev identity broker comm ldap/sts
    - broker auths ldap, then sts
    - app gets temp access to aws resouces

### AD federation

- overview
  - user browsers to adfs sample site in his domain (https://fullyqualifieddomainname.here/adfs/ls/ldpinitiatedsignon.aspx.
  - when you install adfs, you get a new virtual directory names edfs for your default site, which includes the page
  - sign on page auths person against ad, might be prompted for username/pw depending on browser
  - browser receives saml assertion in the form of an authtication response from adfs
  - broser posts assertion to aws signin endpoint for saml
    - behind the scenes, signin uses AssumeRoleWithSAML api to request temp credentials, then constructs a signin url for console
  - browsers gets signin url and is redirected to console

### web identity federation with mobile apps

- for things like fb, goog, etc
- can get testing tools/response from iam/web identity federation playfround
- examples steps
  - 1 auth with identity provider, get back response with
    - access token
    - userid
    - expiesin
    - signedrequest (looks like a cert type thing)
  - 2 obtain temp sec credentials, call AssumeRoleWithWebIdentity with these details
    - trust policy (policy doc with assume role permission)
    - providerid (graph.facebook.com)
    - iam role arn
    - role session name (web-identity-federation)
    - webidentitytoken (token from first step)
  - get response back with session token
  - 3 access resource
    - access policy
    - access key
    - key id
    - session token
    - action

## EC2

- spot instance type - you set bid price. will only provision when the price of ec2 (based on demand) goes below your bid price
- types

| family | speciality                    | use case                            | way to remember                               |
| ------ | ----------------------------- | ----------------------------------- | --------------------------------------------- |
| d2     | dense storage                 | fileserver/data warehousing/hadooop | d for dense                                   |
| r4     | memory opti8mized             | memory intensive apps/dbs           | r for ram                                     |
| m4     | gen purpose                   | app servers                         | m for main (should be default go-to for apps) |
| c4     | compute optimized             | cpu intensive/dbs                   | c for cpu                                     |
| g2     | graphics intensive            | video encoding/3d app steaming      | g for graphics                                |
| i2     | high speed storage            | nosqldb, data wh                    | i for iops                                    |
| f1     | field programmable gate array | hw acceleration for code            |
| t2     | lowest cost, gen purpose      | web servers/smalldb                 |
| p2     | graphics/general purpose gpu  | machine learning/bitcoin mining     | p in gpu or as pics                           |
| x1     | mem optimized                 | sap hana/apache spark               | x for xtreme ram                              |

DR Mc GIFT PX

D dense
R ram
M main, app
C cpu
G gpu
I Iops
F programmable gate
T low tier
P pics/gpu
X extreme

- EBS
  - replicated within availability zone automatically
  - gp2 - general
    - 3 iops per gb w/ up to 10,000 iops and ability to burst to 3000 iops for extended periods if above 3334gb
  - io1 - provisioned iops ssd
    - good for io intense such as relational dbs or nosql
    - use if you need more than 10k iops
    - can go up to 20000
  - st1 - hdd for big data, data warehouses, log processing
    - good for sequential writes
    - cannot be boot volume
  - sc1 - lowest cost storage for file servers
    - cannot be boot
  - magnetic standard - lowest cost of all that is bootable
- Know for exam:
  - difference between on-demand, spot. reserved, dedicated hosts
  - spot - terminate the instance, pay for the hour
    - if aws terminates, you get the hour for free (price has gone above your bid)
  - ebs types
  - cannot mount an ebs to multiple instances
    - use efs instead
  - know ec2 instance types
  - 1 subnet, 1 availability zone
- Security groups
  - cant explicitly deny traffic, it's blocked by default
  - use network ACLs to block individual IPs
- EFS
  - use case is to be able to mount a volume/file system to multiple ec2 instances
  - supports nfsv4
  - no preprovisioning required, only pay for what you use
  - can scale up to petabytes, can support 1000s of concurrent nfs connections
  - across multiple AZs in a region
  - block based storage
  - read after write consistency
- Lambda
  - can be used in the follwoing ways:
    - event-driven compute service where aws lambda run code in response to events, events could be changes to data in s3 or dynamodb, etc
    - as a compute service to run code in response to https equests using amazon api gateway or api calls made using aws sdks
  - example:
    - user creates a meme with text, triggers lambda event 1
    - L1 adds text to image and stores in bucket, then triggers L2
    - L2 returns location of image back to user, then triggers L3
    - L3 copies image to another bucket for backup
  - languages
    - nodejs
    - java
    - python
    - c#
  - billed based on requests and duration of requests
    - rounded up to nearest 100ms
  - version control
    - each lambda function version has a unique arn
    - after you blish a version, it is immutable
    - you can split traffic between versions (blue/green) by using an alias and setting weights for two versions:
      - v3 - weight:78%
      - v2 - weight:22%
      - must set versions explicitly, can't use $Latest
- ELB
  - instances monitored by elb are reported as InService or OutofService
  - healthchecks work by looking for an individual file through http or https
  - have their own dns, never given an ip

## s3

- max size 5tb
- key/value store. objects consist of:
  - key (name of object)
    - s3 designed to be lexographical, store in alphabetical order
      - big design consideration, could be on exam
      - data will be physically stored in same area, so it could cause bottlenecks
      - adding a salt will ensure objects are stored evenly accross s3
  - value (this is just the data and made up of sequence of bytes)
  - versionid (important for versionning)
  - metadata
  - subresources
    - acls
    - torrent (s3 dos support torrent protocol)
- 4 9s availability
- 11 9s durability
- s3 ia charges retrieval fee
- reduced redundancy storage - 4 9s for durability
- 3 - 5 hours to restore from glacier
- enc
  - client side (encrypt before uploading)
  - amz s3 managed keys sse-s3
    - aws manages keys, each object is encrypted with its own key
    - keys are encrypted with a master key that's regularly rotated for you
  - kms sse-kms (keys through kms)
    - aws still manages
    - sep permission for use of additional key called envelope key which encrypts your key
  - customer provided keys sse-c (provide key)
- two ways to control access to buckets
  - acls
    - applied at object level
  - policy
    - applied at bucket level
- CORS cross origin resource sharing
  - allows javascript in one s3 bucket to reference code in another s3 bucket
  - allows one resource to access another
  - policy only allows the EXACT url
- encrypt at upload time by including a header:
  - x-amz-server-side-encryption: AES256
  - x-amz-server-side-encryption: ams:kms
- enforce encryption by using bucket policy which denies s3 put request that doesnt include the enc header
- cloudfront
  - two types of distributions
    - web
    - rtmp (adobe real time messaging protocol) - used for media steaming/flash multi-media content
- s3 performance
  - if s3 buckets are regularly getting >100 PUT/list/delete or >300 GET per sec
    - GET-intensive workloads should use cloudfront
    - mixed (get/put/list/delete), key name makes a difference because of the s3 hardware thing
      - alphabetical, salt names, go read that section
- same-origin policy - web browser permits scripts contained in a first web page to acess data in a second web page, but only if both have the same origin
  - done to prevent cross-site scripting (XSS) attacks.
    - enforced by web browsers
    - ignored by tools like postman/curl
- cross-origin resource sharing (CORS) is a way for the server (not the client code in the browser) to relax the same-origin policy
  - allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served.
- step functions
  - allow you to visualize and test serverless apps
  - graphical console to arrange and visualize the components of an app as a series of steps
  - auto triggifers and trakcks each step, retries when there are errors
  - logs state of each step, for easier debug
  - can be sequential steps or branches of steps, like a flowchart, or parallel steps
  - doesn't actually attach to existing lambda, creates them for you using amazon state language
  - creates using cloudformation, so just delete the stack when not needed anymore
- xray
  - collexts data about requests that app serves and gives tools to view/filter/get insight into that data to identify issues/ways to optimize
  - w/ traced requests, get info about request/response/calls app makes downstream to other resources, microservices, databases, web apis
  - installed as sdk in app, sends info as json to xray daemon (can be used on linux/windows/osx), which sends to xray api
  - aws scripts/tools (cli/sdk) can comm with daemon or xray api
  - sdk gives:
    - interceptors to add to code to trace incoming http requests
    - client handlers to instrument aws sdk clients your app uses to call other aws services
    - http client to instrument calls to other int/ext http web services
  - integrates w/:
    - elb
    - lambda
    - api gateway
    - ec2
    - ebs
  - supported languages:
    - java
    - go
    - node.js
    - python
    - ruby
    - .net

## DynamoDb

- stores everything on ssd
- spread across 3 geographically distinct datacenters
- 2 consistency models
  - eventual consistency reads (default)
    - best perf
    - consistency usually reached within a second
    - repeating a read after a short time should return the updated data
  - strongly consistent reads
    - strongly consistent read returns a result that reflects all writes that received a successful response prior to the read
- made up of:
  - tables
  - items (think a row in the table)
  - attributes (column)
  - supports key-value and document data structures
  - documents can be either json, html, or xml
- stores and retrieves data based on primary key
- 2 types of primary key
  - partition key - unique attribute (eg user id)
    - value value of partition key is input to an internal hash function which determines the partition or physical location on which the data is stored
      - looks like it'll be something like (userid + hash function = key)
    - if using partition key as primary key, no two items can have the same partition key
  - composite key (partition key + sort key)
    - eg same user posting multiple times to a forum
    - primary key would be a composite key consisting of
      - partition key - user id
      - sort key - timestamp of post
    - 2 items may have the same partition key, but must have different sort key (same userid, but different time stamps)
    - all items w/ same partition key are stored together, then sorted according to sort key value
    - allows you to store multiple items w/ same partition key
    - items with same partition key are stored together, than sorted by sort key
- auth managed through iam
- can create permanent role for access or a role that gets temp access keys to access dynamodb
- can also use a special iam condition to restrict user access to only their own records
  - ie game where users need to access their high scores and not anyone elses
    - can be done by adding a confition to an iam policy to allow access only to items where partition key value matches their userid
    - {"Condition": {"ForAllValues:StringEquals": {"dynamodb:LeadingKeys": ["${www.mygame.com:user_id}"]}}}
- two types of indexes supported
  - local secondary
    - can only be created when creating table
    - cannot add/remove/modify later
    - has the same partition key as original table
    - but a different sort key
    - gives different view of data, organized according to alternative sort key
    - any queries based on this sort key are much faster using the index than the mnain table
    - eg partition key:User ID
         Sort Key: account creation date
  - global secondary
    - can create when making table or add later
    - allows you to choose a diff partition key as well as different sort key
    - gives a completely diff view of data
    - speeds up any queries relating to this alternative partition and sort key
    - eg partition key - email address
         sort key - last log-in date
- query finds items in a table based on primary key attr and distinct value to search for
  - select an item where the userd is is equal to 212 will select all attr for that item
  - use an optional sort key name/val to refine results
  - if your sort key is a timestemp, you can refine the query to only select items with a timestamp of last 7 days
  - by default, a query returns all atrr for items, but you can use the projection expression param if you want the query to only return specific attr
    - eg if you only want to see the email addr rather than all the attr
  - results are always sorted by the sort key
  - numeric order - by default in ascending order (1,2,3,4)
  - ascii character code values
  - can reverse order by setting the ScanIndexForward parameter to false
  - by default, queries are eventually consistent
  - you need to explicitly set the query to be strongly consistent
- scan op examines every item in table
  - by default returns all data attr
  - use the ProjectionExpression param to refine the scan to only return attr you want
- query is more efficient than scan
- scan dumps entire table, then filters (like ? number -gt 2)
- adds an extra step of removing the data you dont want
- scans take longer as table grows
- scan operation on a large table can use up the provisioned throughtput ofor a large table in just a single op
- how to improve perf
  - reduce impact of a query or scan by setting smaller page size, which uses fewer read ops
    - eg set page size to return 40 items
  - larger number of smaller ops will allow other requests to succeed w/o throttling
  - avoid using scan if you can: design tables in a way you can use query, get, or batchgetitem apis
  - by default, scan op process data sequentially in returning 1mb increments before moving on to next 1mb. can only scan one partition at a time
  - configure dynamondb to use parallel scans instead by logically dividing a table/index into segments and scanning each segment in parallel
  - best to avoid parallel scans if your table/index is already incurring heavy read/write activity from other apps

### provisioned throughput

- provisioned throughput measured in capacity units
- when you create table, specify requirements in terms of read capacity units and write cpacity units
- 1x write capacity unit = 1 x 1kb write per sec
- 1x read capacity unit =
  - 1 x strong consistent read of 4kb read per sec
  - 2x eventual consistent read of 4kb read per sec

## kms/enc

-

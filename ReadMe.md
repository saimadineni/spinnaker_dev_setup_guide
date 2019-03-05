# Hitchhiker's guide to setup spinnaker development environment

## Step 1: Pre Reqs
a. Ensure that you've java jdk 1.8 setup on your mac.

The latest jdk version is jdk 11 or something. It'll not work. You need to ensure 

**jdk 1.8**

For ref. I've 
`java -version`
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)

And also make sure you export JAVA_HOME, This is needed every time you open a new terminal. So, just add it your bash profile

In my case, Since I've installed jdk1.8.0_201 the path is as mentioned `export JAVA_HOME='/usr/java/jdk1.8.0_201-amd64'
`
b. git

c. curl

d. netcat

e. redis-server// After installing redis start it in the background as well as it'll be used by Clouddriver for caching.

f. node
`  curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
  # Follow instructions at end of script to add nvm to ~/.bash_rc

  nvm install v8.9.0
`

f. yarn `npm install -g yarn`


## Step 2: Fork  all of these github repos and clone them on to your machine/workspace

Service and the corresponding port the service will run on.

| service | port |
| Clouddriver |	7002 |
| Deck |	9000 |
| Echo |	8089 |
| Fiat |	7003 |
| Front50 |	8080 |
| Gate |	8084 |
| Halyard |	8064 |
| Igor |	8088 |
| Kayenta |	8090 |
| Orca |	8083 |
| Rosco |	8087 |

## Step 3: Setup halyard from scratch as it is the main service that needs to run for us to test clouddriver.

navigate to  halyard/halyard-cli directory and enter

command: `make`

That'll generate a local hal binary and few other things.

`cd ..`  and

Run `./gradlew`

now `cd halyard-cli` and run `./hal -v` and now that should return the version of halyard. If not something went wrong with the deployment.


## Step 4: Choose your cloud provider

You can choose either of the options EC2, EKS and ECS listed here

https://www.spinnaker.io/setup/install/providers/aws/

I've chose EC2 and follwed the steps here. You can setup managed/managing infra. both in the same aws account

https://www.spinnaker.io/setup/install/providers/aws/aws-ec2/

` curl https://d3079gxvs8ayeg.cloudfront.net/templates/managing.yaml
echo "Optionally add Managing account to the file downloaded as shown at https://github.com/spinnaker/spinnaker.github.io/tree/master/setup/install/providers/aws/managing.yaml#L104"
aws cloudformation deploy --stack-name spinnaker-managing-infrastructure-setup --template-file managing.yaml \
--parameter-overrides UseAccessKeyForAuthentication=false --capabilities CAPABILITY_NAMED_IAM --region us-west-2`

Before running the below command make sure you replace AuthArn and Managing account details from the output of the above stack.

`curl https://d3079gxvs8ayeg.cloudfront.net/templates/managed.yaml
aws cloudformation deploy --stack-name spinnaker-managed-infrastructure-setup --template-file managed.yaml \
--parameter-overrides AuthArn=FROM_ABOVE ManagingAccountId=FROM_ABOVE --capabilities CAPABILITY_NAMED_IAM --region us-west-2`

Configure Halyard to use AccessKeys (if configured)

`./hal config provider aws edit --access-key-id ${ACCESS_KEY_ID} \
    --secret-access-key `

Configure Halyard to add AWS Accounts

`$AWS_ACCOUNT_NAME={name for AWS account in Spinnaker, e.g. my-aws-account}

./hal config provider aws account add $AWS_ACCOUNT_NAME \
    --account-id ${ACCOUNT_ID} \
    --assume-role role/spinnakerManaged`

Now enable AWS

`./hal config provider aws enable`

## Step 5: Choose your Environment, In this step, you tell Halyard in what type of environment to install Spinnaker.

we'll choose local installations from github

`./hal config deploy edit --type localgit --git-origin-user=<YOUR_GITHUB_USERNAME>

./hal config version edit --version branch:upstream/master`

NOTE: Be sure to use the same username here that you forked the Spinnaker repositories to


## Step 6: Choose S3 as your storage type

`YOUR_SECRET_KEY_ID=your_aws_secret_key_id`
`REGION=YOUR_REGION`

`./hal config storage s3 edit \
    --access-key-id $YOUR_SECRET_KEY_ID \
    --secret-access-key \
    --region $REGION`

Finally, set the storage source to S3:

`hal config storage edit --type s3`

## Step 7: deploy spinnaker

a. List the available versions:

`hal version list`
You can follow the links to the versions’ respective changelogs to see what features each adds.

Set the version you want to use:

`hal config version edit --version $VERSION`

For ref. I've set my VERSION to  1.12.4 by `VERSION= 1.12.4`

And now run `./hal deploy apply` and command will succeed until the end it'll probably fail.

But now can't connect to the hal will start But you can't connect to the ui yet.

## Step 8: Start all the individual services

Except deck, go into all the repo's you have downloaded and execute `./gradlew` command this command will start all the individual services.





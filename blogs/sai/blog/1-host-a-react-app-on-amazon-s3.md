If you are like me, where you have built your own portfolio website or a blog with static content or a static website for your business, using React, the website will only consist of HTML, CSS and Javascript. When you do not have any requirements to run server side code, then there is really no point of hosting your code on a traditional server provided by a traditional hosting provider. 

Amazon S3 is a great option to host the React App as it provides industry-leading scalability, data availability, security, and performance. Also, hosting your React App on Amazon S3 is very cheap.

So how do I host my React App on Amazon S3?

First, you would need to create an AWS account (if you don't already have one). You can do that from the [AWS signup portal](https://portal.aws.amazon.com/billing/signup#/start).

#### Step 1: IAM
  * Once the account is up and running, click on "Services" and search for "IAM" and click on it to open the Identity and Access Management console.
  * It is [security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) to lock away your root user access to your account. 
  * Hence, we'll create a "User" in the account with necessary permissions to to push our React App to S3.
    * To create a user, select the "Users" option under the "Access management" section of the IAM dashboard.
    * Click on the "Add user" button.
    * Choose a user name.
    * Under "Access type", select "Programmatic access" for the user so that the user can easily sync the React app to S3 using AWS CLI. Also, select "AWS Management Console access" so that the user has access to the AWS console and then click on the "Next: Permissions" button at the bottom.
    * In the "Set permissions" page, you can attach an existing policy like "AmazonS3FullAccess" or "AdministratorAccess" to the user OR click on the "Next: Tags" button and we can associate an IAM policy after we finish the user creation process.
    * Add tags to the user (optional). Click on "Next: Review".
    * If you have not attached any policies to the user, you'll see a warning which says "This user has no permission". You can ignore the warning and create the user.
    * After the user is created, you'll see a page with "Access key ID" and "Secret access key". You need to store thes values safely so that your account is not misused. You can also download these keys in a CSV file by clicking on the "Download .csv" button or send the login instructions via email.
    * Save the "Access key ID" and "Secret access key" details somewhere safe. We'll use these later when setting up the AWS CLI.
  * If you have not already attached a suitable IAM policy to the user, when creating the user, below are the steps to create a new policy or attach an exisiting policy, suitable to the tasks performed by the user.
    * IAM policies can be used to control the permissions of the user.
    * When you click on "Policies" under the "Access management" section of the IAM dashboard, you'll see a list of available policies.
    * You can either choose from the list of available policies and attach suitable policies to the user or create a new policy and attach the newly created policy to the user.
    * You can refer to [user policy examples](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-policies-s3.html) for controlling a user's access to S3. 
    * For the purposes of this blog, I have attached the user with "AmazonS3FullAccess" managed policy.

&nbsp

#### Step 2: Setup AWS CLI
  * If you do not have AWS CLI installed on your computer:
    * The easiest way to install AWS CLI is by using [pip](https://pypi.org/project/awscli/) if you already have python and pip installed
    ```bash
    $ python -m pip install awscli
    ```
    * You can also follow AWS's documentation on [how to install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html).
  * Before you can use the AWS CLI, you would need to configure it to interact with AWS and make API calls on a user's/roles's behalf. The configuration includes the user credentials (the access key and the secret access key discussed above in the IAM section), default AWS region that you are woking in and the default output format. 
  * The quickest way to do this is to use the ```aws configure``` command
  ```
  $ aws configure
  AWS Access Key ID [None]: <Enter the access key of the user>
  AWS Secret Access Key [None]: <Enter the secret access key of the user>
  Default region name [None]: <Enter an AWS region ex: ap-southeast-2>
  Default output format [None]: json
  ```
  * AWS CLI uses a set of credential providers to look for AWS credentials. You can refer to AWS's documentation on [configuration settings and precedence](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#config-settings-and-precedence) for more information.

#### Step 3: Create an S3 bucket and configure it for static web hosting
To create a bucket:
  * Login to AWS and go to the [S3 console](https://console.aws.amazon.com/s3/). 
  * An S3 bucket name is globally unique and the namespace is shared by all AWS accounts. So common bucket names like "test", "bucket" or "website" may not be available.
  * If you already have a domain registered, you can try using your domain name for your bucket name. Ex: mydomain.com
  * Click on the "Create Bucket" button, enter a unique bucket name and hit the "Create Bucket" button. You can leave the rest of the configuration as default.

To setup static web hosting on the S3 bucket:
  * Once the bucket is created, select the bucket in the S3 console and click on the "Properties" tab. 
  * Click on the "Static website hosting" card and select the "Use this bucket to host a website" option. In the "Index document" text field, enter the name of your index document. This is generally index.html
  * Note the endpoint name which is on the top of the "Static website hosting card".
  * Hit Save.

Set permissions so that people can access your website:
  * When you select the bucket, click on the "Permissions" tab. You'll notice "Block all public access" as On.
  * Click on the edit button and uncheck the "Block all public access" option and save.
  * You'll see a pop up asking to confirm the changes. Type confirm in the text field and select the confirm button.
  * Under the same "Permissions" tab, select the "Bucket Policy" option.
  * Copy and paste the below policy, change <your-bucket-name> to your bucket name and save the policy.
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "PublicReadGetObject",
        "Effect": "Allow",
        "Principal": "*",
        "Action": [
            "s3:GetObject"
        ],
        "Resource": [
            "arn:aws:s3:::<your-bucket-name>/*"
        ]
      }
    ]
  }
  ```
  * Now go back to the "Block public access" section and click edit.
  * Except for the last option "Block public and cross-account access to buckets and objects through any public bucket or access point policies", block other access by selecting the other 3 options and save.

Your bucket should now be accessible for the public to view your website on the bucket endpoint URL. You can access this URL when you select the "Static website hosting" card under the "Properties" tab.

#### Step 4: Deploying your React App to S3 for the world to see

If you are reading this blog, you must have already created a React app on your computer but in case you are just starting out and have not yet created the React app on your computer, you can refer to the [Create React App](https://create-react-app.dev/docs/getting-started) documentation, which is very detailed.

I have used yarn to create my react app:
```bash
$ yarn create react-app mywebsite
$ cd mywebsite
$ yarn start
```

Open the React app folder in a text editor like Visual Studio Code. If you list the contents of the folder, you should see the "package.json" file.

Open the "package.json" file in the text editor. Under the scripts section of the file, you can add an AWS CLI command to sync your build folder to the S3 bucket.

```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject",
  "push": "aws s3 sync build/ s3://<your-bucket-name>/",
}
```

I have added the S3 sync command as a script called "push". Now, in order to sync the build folder to S3, first make sure you are in the same directory as the react app folder. After you have saved all the changes you have made in your React app locally, you can run the below command to build and push your website to S3:
```bash
$ yarn build && yarn push
```

&nbsp

Your website should now be pushed to S3 and should be available for the world to see at the bucket endpoint url: 
http://<your-bucket>.s3-website-<region>.amazonaws.com





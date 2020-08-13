> Prerequisite: [Install and configure](~/cli/start/install.md) the Amplify CLI

## Storage with Amplify

AWS Amplify Storage module provides a simple mechanism for managing user content for your app in public, protected or private storage buckets. The Storage category comes with built-in support for Amazon S3.

## Automated Setup: Create storage bucket

To start from scratch, run the following command from the root of your project:

```bash
amplify add storage
```

and select *Content* in prompted options:

```console
? Please select from one of the below mentioned services (Use arrow keys)
❯ Content (Images, audio, video, etc.)
  NoSQL Database
```

The CLI will walk you though the options to enable Auth, if not enabled previously, and name your S3 bucket. To update your backend run:

```bash
amplify push
```

When your backend is successfully updated, your new configuration file `aws-exports.js` is copied under your source directory, e.g. '/src'.

## Configure your application

Add Amplify to your app with `yarn` or `npm`:

```bash
npm install -S aws-amplify
```

In your app’s entry point i.e. `App.js`, import and load the configuration file `aws-exports.js` which has been created and replaced into `/src` folder in the previous step.
```javascript
import Amplify, { Storage } from 'aws-amplify';
import awsconfig from './aws-exports';
Amplify.configure(awsconfig);
```

## Manual Setup: Import storage bucket

If you use `aws-exports.js` file, Storage is already configured when you call `Amplify.configure(awsconfig)`. To configure Storage manually, you will have to configure Amplify Auth category as well. 

Manual setup enables you to use your existing Amazon Cognito and Amazon S3 credentials in your app:

```javascript
import Amplify, { Auth, Storage } from 'aws-amplify';

Amplify.configure({
    Auth: {
        identityPoolId: 'XX-XXXX-X:XXXXXXXX-XXXX-1234-abcd-1234567890ab', //REQUIRED - Amazon Cognito Identity Pool ID
        region: 'XX-XXXX-X', // REQUIRED - Amazon Cognito Region
        userPoolId: 'XX-XXXX-X_abcd1234', //OPTIONAL - Amazon Cognito User Pool ID
        userPoolWebClientId: 'XX-XXXX-X_abcd1234', //OPTIONAL - Amazon Cognito Web Client ID
    },
    Storage: {
        AWSS3: {
            bucket: '', //REQUIRED -  Amazon S3 bucket name
            region: 'XX-XXXX-X', //OPTIONAL -  Amazon service region
        }
    }
});

```

### Using Amazon S3
If you set up your Cognito resources manually, the roles will need to be given permission to access the S3 bucket.

There are two roles created by Cognito: an `Auth_Role` that grants signed-in-user-level bucket access and an `Unauth_Role` that allows unauthenticated access to resources. Attach the corresponding policies to each role for proper S3 access. Replace ```{enter bucket name}``` with the correct S3 bucket.

Inline policy for the `Auth_Role`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}/public/*",
                "arn:aws:s3:::{enter bucket name}/protected/${cognito-identity.amazonaws.com:sub}/*",
                "arn:aws:s3:::{enter bucket name}/private/${cognito-identity.amazonaws.com:sub}/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}/uploads/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}/protected/*"
            ],
            "Effect": "Allow"
        },
        {
            "Condition": {
                "StringLike": {
                    "s3:prefix": [
                        "public/",
                        "public/*",
                        "protected/",
                        "protected/*",
                        "private/${cognito-identity.amazonaws.com:sub}/",
                        "private/${cognito-identity.amazonaws.com:sub}/*"
                    ]
                }
            },
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}"
            ],
            "Effect": "Allow"
        }
    ]
}
```

Inline policy for the `Unauth_Role`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}/public/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}/uploads/*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}/protected/*"
            ],
            "Effect": "Allow"
        },
        {
            "Condition": {
                "StringLike": {
                    "s3:prefix": [
                        "public/",
                        "public/*",
                        "protected/",
                        "protected/*"
                    ]
                }
            },
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::{enter bucket name}"
            ],
            "Effect": "Allow"
        }
    ]
}
```

The policy template that Amplify CLI uses is found [here](https://github.com/aws-amplify/amplify-cli/blob/b12d20b9d85f7fc6abf7e2f7fbe11e1a108911b9/packages/amplify-category-storage/provider-utils/awscloudformation/cloudformation-templates/s3-cloudformation-template.json).

### Amazon S3 Bucket CORS Policy Setup

<amplify-callout warning>

To make calls to your S3 bucket from your App, you need to set up a CORS Policy for your S3 bucket. This callout is only for manual configuration of your S3 bucket, CORS Policy configuration is done automatically via Amplify CLI when running `amplify add storage`.

</amplify-callout>

The following steps will set up your CORS Policy: 

1. Go to [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home?region=us-east-1) and click on your project's `userfiles` bucket, which is normally named as [Project Name]-userfiles-mobilehub-[App Id]. 
2. Click on the **Permissions** tab for your bucket, and then click on the **CORS configuration** tile.
3. Update your bucket's CORS Policy to look like:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>DELETE</AllowedMethod>
    <MaxAgeSeconds>3000</MaxAgeSeconds>
    <ExposeHeader>x-amz-server-side-encryption</ExposeHeader>
    <ExposeHeader>x-amz-request-id</ExposeHeader>
    <ExposeHeader>x-amz-id-2</ExposeHeader>
    <ExposeHeader>ETag</ExposeHeader>
    <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

<amplify-callout>

**Note:** You can restrict the access to your bucket by updating AllowedOrigin to include individual domains.

</amplify-callout>

For information on Amazon S3 file access levels, please see [file access levels](~/lib/storage/configureaccess.md).

## Using a Custom Plugin

You can create your custom pluggable for Storage. This may be helpful if you want to integrate your app with a custom storage backend.

To create a plugin implement the `StorageProvider` interface:

```typescript
import { Storage, StorageProvider } from 'aws-amplify';

export default class MyStorageProvider implements StorageProvider {
    // category and provider name
    static category = 'Storage';
    static providerName = 'MyStorage';

    // you need to implement these seven methods
    // configure your provider
    configure(config: object): object;

    // get object/pre-signed url from storage
    get(key: string, options?): Promise<String|Object>

    // upload storage object
    put(key: string, object, options?): Promise<Object>

    // remove object 
    remove(key: string, options?): Promise<any>

    // list objects for the path
    list(path, options?): Promise<any>
    
    // return 'Storage';
    getCategory(): string;
    
    // return the name of you provider
    getProviderName(): string;
```

You can now register your pluggable:

```javascript
// add the plugin
Storage.addPluggable(new MyStorageProvider());

// get the plugin
Storage.getPluggable(MyStorageProvider.providerName);

// remove the plugin
Storage.removePluggable(MyStorageProvider.providerName);

// send configuration into Amplify
Storage.configure({
    [MyStorageProvider.providerName]: { 
        // My Storage provider configuration 
    }
});

```

The default provider (Amazon S3) is in use when you call `Storage.put( )` unless you specify a different provider: `Storage.put(key, object, {provider: 'MyStorageProvider'})`. 


## Mocking and Local Testing with Amplify CLI
Amplify CLI supports running a lock mock server for testing your application with Amazon S3. Please see the [CLI toolchain documentation](~/cli/usage/mock.md) for more details.

## API Reference

For the complete API documentation for Storage module, visit our [API Reference](https://aws-amplify.github.io/amplify-js/api/classes/storageclass.html).
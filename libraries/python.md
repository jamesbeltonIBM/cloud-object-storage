---

copyright:
  years: 2017, 2018
lastupdated: "2018-08-24"

---

# Using Python

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}

Python support is provided through a fork of the Boto library.  It can be installed from the Python Package Index via `pip install ibm-cos-sdk`.

Source code can be found at [GitHub](https://github.com/ibm/ibm-cos-sdk-python/).

The `ibm_boto3` library provides complete access to the {{site.data.keyword.cos_full}} API.  Endpoints, an API key, and the instance ID must be specified when creating a service resource or low-level client as shown in the following basic examples.

The service instance ID is also referred to as a _resource instance ID_.  The value can be found by creating a [service credential](/docs/services/cloud-object-storage/iam/service-credentials.html), or through the CLI.
{:tip}

Detailed documentation can be found at [here](https://ibm.github.io/ibm-cos-sdk-python/).

## Migrating from 1.x.x
The 2.0 release of the SDK introduces a namespacing change that allows an application to make use of the original `boto3` library to connect to AWS resources within the same application or environment.  To migrate from 1.x to 2.0 some changes are necessary.

    1. Update the `requirements.txt`, or from PyPI via `pip install -U ibm-cos-sdk`.  Verify no older versions exist with `pip list | grep ibm-cos`.
    2. Update any import declarations from `boto3` to `ibm_boto3`.
    3. If needed, reinstall the original `boto3` by updating the `requirements.txt`, or from PyPI via `pip install boto3`.

## Creating a client and sourcing credentials
{: #client-credentials}

To connect to COS, a client is created and configured by providing credential information (API key and service instance ID). These values can also be automatically sourced from a credentials file or from environment variables.  

After generating a [Service Credential](/docs/services/cloud-object-storage/iam/service-credentials.html), the resulting JSON document can be saved to `~/.bluemix/cos_credentials`.  The SDK will automatically source credentials from this file unless other credentials are explicitly set during client creation. If the `cos_credentials` file contains HMAC keys the client will authenticate with a signature, otherwise the client will use the provided API key to authenticate using a bearer token.

If migrating from AWS S3, you can also source credentials data from  `~/.aws/credentials` in the format:

```
[default]
aws_access_key_id = {API_KEY}
aws_secret_access_key = {SERVICE_INSTANCE_ID}
```

If both `~/.bluemix/cos_credentials` and `~/.aws/credentials` exist, `cos_credentials` will take preference.

## Code Examples

Code examples were written using **Python 2.7.15**

### Initializing configuration
{: #init-config}

```python
# Constants for IBM COS values
COS_ENDPOINT = "<endpoint>"
COS_API_KEY_ID = "<api-key>"
COS_AUTH_ENDPOINT = "https://iam.ng.bluemix.net/oidc/token"
COS_SERVICE_CRN = "<resource-instance-id>"
COS_BUCKET_LOCATION = "<location"

# Create resource
cos = ibm_boto3.resource("s3",
    ibm_api_key_id=COS_API_KEY_ID,
    ibm_service_instance_id=COS_SERVICE_CRN,
    ibm_auth_endpoint=COS_AUTH_ENDPOINT,
    config=Config(signature_version="oauth"),
    endpoint_url=COS_ENDPOINT
)
```
*Key Values*
* `<endpoint>` - public endpoint for your cloud object storage (available from the [IBM Cloud Dashboard](https://console.bluemix.net/dashboard/apps){:new_window})
* `<api-key>` - api key generated when creating the service credentials (write access is required for creation and deletion examples)
* `<resource-instance-id>` - resource ID for your cloud object storage (available through [IBM Cloud CLI](../getting-started-cli.html) or [IBM Cloud Dashboard](https://console.bluemix.net/dashboard/apps){:new_window})
* `<location>` - default location for your cloud object storage (must match the region used for `<endpoint>`)

*SDK References*
* [ServiceResource](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#service-resource){:new_window}

### Creating a new bucket
```python
def create_bucket(bucket_name):
    print("Creating new bucket: {0}".format(bucket_name))
    try:
        cos.Bucket(bucket_name).create(
            CreateBucketConfiguration={
                "LocationConstraint":COS_BUCKET_LOCATION
            }
        )
        print("Bucket: {0} created!".format(bucket_name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to create bucket: {0}".format(e))
```

Valid provisioning codes for `LocationConstraint` are: <br>
&emsp;&emsp;  `us-standard` / `us-vault` / `us-cold` / `us-flex` <br>
&emsp;&emsp;  `us-east-standard` / `us-east-vault`  / `us-east-cold` / `us-east-flex` <br>
&emsp;&emsp;  `us-south-standard` / `us-south-vault`  / `us-south-cold` / `us-south-flex` <br>
&emsp;&emsp;  `eu-standard` / `eu-vault` / `eu-cold` / `eu-flex` <br>
&emsp;&emsp;  `eu-gb-standard` / `eu-gb-vault` / `eu-gb-cold` / `eu-gb-flex` <br>
&emsp;&emsp;  `eu-de-standard` / `eu-de-vault` / `eu-de-cold` / `eu-de-flex` <br>
&emsp;&emsp;  `ap-standard` / `ap-vault` / `ap-cold` / `ap-flex` <br>
&emsp;&emsp;  `ams03-standard` / `ams03-vault` / `ams03-cold` / `ams03-flex` <br>
&emsp;&emsp;  `che01-standard` / `che01-vault` / `che01-cold` / `che01-flex` <br>
&emsp;&emsp;  `mel01-standard` / `mel01-vault` / `mel01-cold` / `mel01-flex` <br>
&emsp;&emsp;  `osl01-standard` / `osl01-vault` / `osl01-cold` / `osl01-flex` <br>
&emsp;&emsp;  `tor01-standard` / `tor01-vault` / `tor01-cold` / `tor01-flex` <br>

*SDK References*
* Classes
  * [Bucket](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#bucket){:new_window}
* Methods
    * [create](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Bucket.create){:new_window}

### Creating a new text file
```python
def create_text_file(bucket_name, item_name, file_text):
    print("Creating new item: {0}".format(item_name))
    try:
        cos.Object(bucket_name, item_name).put(
            Body=file_text
        )
        print("Item: {0} created!".format(item_name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to create text file: {0}".format(e))
```

*SDK References*
* Classes
    * [Object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#object){:new_window}
* Methods
    * [put](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Object.put){:new_window}

### List available buckets
```python
def get_buckets():
    print("Retrieving list of buckets")
    try:
        buckets = cos.buckets.all()
        for bucket in buckets:
            print("Bucket Name: {0}".format(bucket.name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to retrieve list buckets: {0}".format(e))
```

*SDK References*
* Classes
    * [Bucket](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#bucket){:new_window}
    * [ServiceResource](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#service-resource){:new_window}
* Collections
    * [buckets](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.ServiceResource.buckets){:new_window}

### List items in a bucket
```python
def get_bucket_contents(bucket_name):
    print("Retrieving bucket contents from: {0}".format(bucket_name))
    try:
        files = cos.Bucket(bucket_name).objects.all()
        for file in files:
            print("Item: {0} ({1} bytes).".format(file.key, file.size))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to retrieve bucket contents: {0}".format(e))
```

*SDK References*
* Classes
    * [Bucket](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#bucket){:new_window}
    * [ObjectSummary](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#objectsummary){:new_window}
* Collections
    * [objects](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Bucket.objects){:new_window}

### Get file contents of particular item
```python
def get_item(bucket_name, item_name):
    print("Retrieving item from bucket: {0}, key: {1}".format(bucket_name, item_name))
    try:
        file = cos.Object(bucket_name, item_name).get()
        print("File Contents: {0}".format(file["Body"].read()))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to retrieve file contents: {0}".format(e))
```

*SDK References*
* Classes
    * [Object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#object){:new_window}
* Methods
    * [get](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Object.get){:new_window}

### Delete an item from a bucket
```python
def delete_item(bucket_name, item_name):
    print("Deleting item: {0}".format(item_name))
    try:
        cos.Object(bucket_name, item_name).delete()
        print("Item: {0} deleted!".format(item_name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to delete item: {0}".format(e))
```

*SDK References*
* Classes
    * [Object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#object){:new_window}
* Methods
    * [delete](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Object.delete){:new_window}

### Delete a bucket
```python
def delete_bucket(bucket_name):
    print("Deleting bucket: {0}".format(bucket_name))
    try:
        cos.Bucket(bucket_name).delete()
        print("Bucket: {0} deleted!".format(bucket_name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to delete bucket: {0}".format(e))
```

*SDK References*
* Classes
    * [Bucket](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#bucket){:new_window}
* Methods
    * [delete](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Bucket.delete){:new_window}

### View a bucket's security
```python
def get_bucket_acl(bucket_name):
    print("Retrieving ACL for bucket: {0}".format(bucket_name))
    try:
        acl_data = cos.BucketAcl(bucket_name)
        print("Owner: {0}".format(acl_data.owner["DisplayName"]))
        for grant in acl_data.grants:
            display_name = grant["Grantee"]["DisplayName"]
            permission = grant["Permission"]
            print("User: {0} ({1})".format(display_name, permission))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to retrieve bucket ACL: {0}".format(e))
```

*SDK References*
* Classes
    * [BucketAcl](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#bucketacl){:new_window}
* Attributes
    * [grants](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.BucketAcl.grants){:new_window}
    * [owner](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.ObjectAcl.owner){:new_window}

### View a file's security
```python
def get_item_acl(bucket_name, item_name):
    print("Retrieving ACL for {0} from bucket: {1}".format(item_name, bucket_name))
    try:
        acl_data = cos.ObjectAcl(bucket_name, item_name)
        print("Owner: {0}".format(acl_data.owner["DisplayName"]))
        for grant in acl_data.grants:
            display_name = grant["Grantee"]["DisplayName"]
            permission = grant["Permission"]
            print("User: {0} ({1})".format(display_name, permission))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to retrieve item ACL: {0}".format(e))
```

*SDK References*
* Classes
    * [ObjectAcl](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#objectacl){:new_window}
* Attributes
    * [grants](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.ObjectAcl.grants){:new_window}
    * [owner](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.ObjectAcl.owner){:new_window}

### Execute a multi-part upload
{: #multipart-upload}

#### Upload binary file (preferred method)
The [upload_fileobj](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Object.upload_fileobj){:new_window} method of the [S3.Object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#object){:new_window} class automatically executes a multi-part upload when necessary.  The [TransferConfig](https://ibm.github.io/ibm-cos-sdk-python/reference/customizations/s3.html#s3-transfers){:new_window} class is used to determine the threshold for using the mult-part upload.

```python
def multi_part_upload(bucket_name, item_name, file_path):
    try:
        print("Starting file transfer for {0} to bucket: {1}\n".format(item_name, bucket_name))
        # set 5 MB chunks
        part_size = 1024 * 1024 * 5

        # set threadhold to 15 MB
        file_threshold = 1024 * 1024 * 15

        # set the transfer threshold and chunk size
        transfer_config = ibm_boto3.s3.transfer.TransferConfig(
            multipart_threshold=file_threshold,
            multipart_chunksize=part_size
        )
        
        # the upload_fileobj method will automatically execute a multi-part upload 
        # in 5 MB chunks for all files over 15 MB
        with open(file_path, "rb") as file_data:
            cos.Object(bucket_name, item_name).upload_fileobj(
                Fileobj=file_data,
                Config=transfer_config
            )
        
        print("Transfer for {0} Complete!\n".format(item_name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to complete multi-part upload: {0}".format(e))
```

*SDK References*
* Classes
    * [Object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#object){:new_window}
    * [TransferConfig](https://ibm.github.io/ibm-cos-sdk-python/reference/customizations/s3.html#s3-transfers){:new_window}
* Methods
    * [upload_fileobj](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Object.upload_fileobj){:new_window}

#### Manually execute a multi-part upload
If desired, the [S3.Client](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window} class can be used to perform a multi-part upload.  This can be useful if more control over the upload process is necessary.

```python
def multi_part_upload_manual(bucket_name, item_name, file_path):
    try:
        # create client object
        cos_cli = ibm_boto3.client("s3",
            ibm_api_key_id=COS_API_KEY_ID,
            ibm_service_instance_id=COS_SERVICE_CRN,
            ibm_auth_endpoint=COS_AUTH_ENDPOINT,
            config=Config(signature_version="oauth"),
            endpoint_url=COS_ENDPOINT
        )
    
        print("Starting multi-part upload for {0} to bucket: {1}\n".format(item_name, bucket_name))

        # initiate the multi-part upload
        mp = cos_cli.create_multipart_upload(
            Bucket=bucket_name,
            Key=item_name
        )

        upload_id = mp["UploadId"]

        # min 5MB part size
        part_size = 1024 * 1024 * 5
        file_size = os.stat(file_path).st_size
        part_count = int(math.ceil(file_size / float(part_size)))
        data_packs = []
        position = 0
        part_num = 0

        # begin uploading the parts
        with open(file_path, "rb") as file:
            for i in range(part_count):
                part_num = i + 1
                part_size = min(part_size, (file_size - position))

                print("Uploading to {0} (part {1} of {2})".format(item_name, part_num, part_count))

                file_data = file.read(part_size)

                mp_part = cos_cli.upload_part(
                    Bucket=bucket_name,
                    Key=item_name,
                    PartNumber=part_num,
                    Body=file_data,
                    ContentLength=part_size,
                    UploadId=upload_id
                )

                data_packs.append({
                    "ETag":mp_part["ETag"],
                    "PartNumber":part_num
                })

                position += part_size

        # complete upload
        cos_cli.complete_multipart_upload(
            Bucket=bucket_name,
            Key=item_name,
            UploadId=upload_id,
            MultipartUpload={
                "Parts": data_packs
            }
        )
        print("Upload for {0} Complete!\n".format(item_name))
    except ClientError as be:
        # abort the upload
        cos_cli.abort_multipart_upload(
            Bucket=bucket_name,
            Key=item_name,
            UploadId=upload_id            
        )
        print("Multi-part upload aborted for {0}\n".format(item_name))
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to complete multi-part upload: {0}".format(e))
```

*SDK References*
* Classes
    * [S3.Client](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}
* Methods
    * [abort_multipart_upload](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Client.abort_multipart_upload){:new_window}
    * [complete_multipart_upload](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Client.complete_multipart_upload){:new_window}
    * [create_multipart_upload](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Client.create_multipart_upload){:new_window}
    * [upload_part](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Client.upload_part){:new_window}

### Large Object Upload using TransferManager
{: #transfer-manager}

The `TransferManager` provides another way to execute large file transfers by automatically incorporating multi-part uploads whenever necessary setting configuration parameters.

```python
def upload_large_file(bucket_name, item_name, file_path):
    print("Starting large file upload for {0} to bucket: {1}".format(item_name, bucket_name))

    # set the chunk size to 5 MB
    part_size = 1024 * 1024 * 5

    # set threadhold to 5 MB
    file_threshold = 1024 * 1024 * 5

    # Create client connection
    cos_cli = ibm_boto3.client("s3",
        ibm_api_key_id=COS_API_KEY_ID,
        ibm_service_instance_id=COS_SERVICE_CRN,
        ibm_auth_endpoint=COS_AUTH_ENDPOINT,
        config=Config(signature_version="oauth"),
        endpoint_url=COS_ENDPOINT
    )

    # set the transfer threshold and chunk size in config settings
    transfer_config = ibm_boto3.s3.transfer.TransferConfig(
        multipart_threshold=file_threshold,
        multipart_chunksize=part_size
    )

    # create transfer manager
    transfer_mgr = ibm_boto3.s3.transfer.TransferManager(cos_cli, config=transfer_config)

    try:
        # initiate file upload
        future = transfer_mgr.upload(file_path, bucket_name, item_name)

        # wait for upload to complete
        future.result()

        print ("Large file upload complete!")
    except Exception as e:
        print("Unable to complete large file upload: {0}".format(e))
    finally:
        transfer_mgr.shutdown()
```

## List items in a bucket (v2)
{: #list-objects-v2}

The [S3.Client](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window} object contains an updated method to list the contents ([list_objects_v2](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Client.list_objects_v2){:new_window}).  This method allows you to limit the number of records returned and retrieve the records in batches.  This could be useful for paging your results within an application and improve performance.

```python
def get_bucket_contents_v2(bucket_name, max_keys):
    print("Retrieving bucket contents from: {0}".format(bucket_name))
    try:
        # create client object
        cos_cli = ibm_boto3.client("s3",
            ibm_api_key_id=COS_API_KEY_ID,
            ibm_service_instance_id=COS_SERVICE_CRN,
            ibm_auth_endpoint=COS_AUTH_ENDPOINT,
            config=Config(signature_version="oauth"),
            endpoint_url=COS_ENDPOINT

        more_results = True
        next_token = ""

        while (more_results):
            response = cos_cli.list_objects_v2(Bucket=bucket_name, MaxKeys=max_keys, ContinuationToken=next_token)
            files = response["Contents"]
            for file in files:
                print("Item: {0} ({1} bytes).".format(file["Key"], file["Size"]))
            
            if (response["IsTruncated"]):
                next_token = response["NextContinuationToken"]
                print("...More results in next batch!\n")
            else:
                more_results = False
                next_token = ""

        log_done()
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to retrieve bucket contents: {0}".format(e))
```

*SDK References*
* Classes
    * [S3.Client](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}
* Methods
    * [list_objects_v2](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Client.list_objects_v2){:new_window}

## Using Key Protect

Key Protect can be added to a storage bucket to encrypt sensitive data at rest in the cloud.

### Before You Begin

The following items are necessary in order to create a bucket with Key-Protect enabled:

* A Key Protect service [provisioned](/docs/services/keymgmt/keyprotect_provision.html)
* A Root key available (either [generated](/docs/services/keymgmt/keyprotect_create_root.html#create_root_keys) or [imported](/docs/services/keymgmt/keyprotect_import_root.html#import_root_keys))

### Retrieving the Root Key CRN

1. Retrieve the [instance ID](/docs/services/keymgmt/keyprotect_authentication.html#retrieve_instance_ID) for your Key Protect service
2. Use the [Key Protect API](/docs/services/keymgmt/keyprotect_authentication.html#access-api) to retrieve all your [available keys](/docs/services/keymgmt/keyprotect_authentication.html#form_api_request)
    * You can either use `curl` commands or an API REST Client such as [Postman](../api-reference/postman.html) to access the [Key Protect API](/docs/services/keymgmt/keyprotect_authentication.html#access-api).
3. Retrieve the CRN of the root key you will use to enabled Key Protect on the your bucket.  The CRN will look similar to below:

`crn:v1:bluemix:public:kms:us-south:a/3d624cd74a0dea86ed8efe3101341742:90b6a1db-0fe1-4fe9-b91e-962c327df531:key:0bg3e33e-a866-50f2-b715-5cba2bc93234`

### Creating a bucket with Key Protect enabled
```python
COS_KP_ALGORITHM = "<algorithm>"
COS_KP_ROOTKEY_CRN = "<root-key-crn>"

# Create a new bucket with key protect (encryption)
def create_bucket_kp(bucket_name):
    print("Creating new encrypted bucket: {0}".format(bucket_name))
    try:
        cos.Bucket(bucket_name).create(
            CreateBucketConfiguration={
                "LocationConstraint":COS_BUCKET_LOCATION
            },
            IBMSSEKPEncryptionAlgorithm=COS_KP_ALGORITHM,
            IBMSSEKPCustomerRootKeyCrn=COS_KP_ROOTKEY_CRN
        )
        print("Encrypted Bucket: {0} created!".format(bucket_name))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        print("Unable to create encrypted bucket: {0}".format(e))
```

*Key Values*
* `<algorithm>` - The encryption algorithm used for new objects added to the bucket (Default is AES256).
* `<root-key-crn>` - CRN of the Root Key obtained from the Key Protect service.

Valid provisioning codes for `LocationConstraint` with Key Protect: <br>
&emsp;&emsp;  `us-south-standard` / `us-south-vault`  / `us-south-cold` / `us-south-flex` <br>
&emsp;&emsp;  `eu-gb-standard` / `eu-gb-vault` / `eu-gb-cold` / `eu-gb-flex` <br>
&emsp;&emsp;  `eu-de-standard` / `eu-de-vault` / `eu-de-cold` / `eu-de-flex` <br>

*SDK References*
* Classes
    * [Bucket](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#bucket){:new_window}
* Methods
    * [create](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#S3.Bucket.create){:new_window}

## Using Archive Tiering

Archive Tier allows users to archive stale data and reduce their storage costs. Archival policies (also known as *Lifecycle Configurations*) are created for buckets and applies to any objects added to the bucket after the policy is created.

### View a bucket's lifecycle configuration
```python
def get_bucket_lifecycle_config(bucket_name):
    print("Retrieving bucket lifecycle config from: {0}".format(bucket_name))
    
    try:
        response = cos_cli.get_bucket_lifecycle_configuration(Bucket=bucket_name)

        print(json.dumps(response["Rules"], indent=4, sort_keys=True))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        log_error("Unable to retrieve bucket lifecycle config: {0}".format(e))
```

*SDK References*
* [get_bucket_lifecycle_configuration](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}

### Create a lifecycle configuration 

Detailed information about structuring the lifecycle configuration rules are available in the [API Reference](/docs/services/cloud-object-storage/api-reference/api-reference-buckets.html#create-bucket-lifecycle)

```python
def create_bucket_lifecycle_config(bucket_name):
    print("Creating bucket lifecycle config for: {0}".format(bucket_name))
    try:
        lifecycle_config = {
            'Rules': [
                {
                    'Filter': {
                        'Prefix': ''
                    },
                    'ID': '<policy-id>',
                    'Status': 'ENABLED',
                    'Transitions': [
                        {
                            'Days': <number-of-days>,
                            'StorageClass': 'GLACIER'
                        }
                    ]
                }
            ]        
        }

        response = cos_cli.put_bucket_lifecycle_configuration(
            Bucket=bucket_name, 
            LifecycleConfiguration=lifecycle_config)
        
        print(json.dumps(response, indent=4, sort_keys=True))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        log_error("Unable to retrieve bucket lifecycle config: {0}".format(e))
```

*Key Values*
* `<policy-id>` - Name of the lifecycle policy (must be unqiue)
* `<number-of-days>` - Number of days to keep the restored file

*SDK References*
* [put_bucket_lifecycle_configuration](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}

### Delete a bucket's lifecycle configuration
```python
def delete_bucket_lifecycle_config(bucket_name):
    print("Deleting bucket lifecycle config from: {0}".format(bucket_name))
    try:
        response = cos_cli.delete_bucket_lifecycle(Bucket=bucket_name)

        print(json.dumps(response, indent=4, sort_keys=True))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        log_error("Unable to retrieve bucket lifecycle config: {0}".format(e))
```

*SDK References*
* [delete_bucket_lifecycle](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}

### Temporarily restore an object

Detailed information about the restore request parameters are available in the [API Reference](/docs/services/cloud-object-storage/api-reference/api-reference-objects.html#restore-object)

```python
def restore_archive_object(bucket_name, item_name):
    print("Restoring item: {0} from bucket: {1}".format(item_name, bucket_name))
    try:
        restore_request = {
            "Days": <number-of-days>, 
            "GlacierJobParameters": {
                "Tier": "Bulk" 
            }
        }

        response = cos_cli.restore_object(
            Bucket=bucket_name, 
            Key=item_name, 
            RestoreRequest=restore_request)

        print(json.dumps(response, indent=4, sort_keys=True))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        log_error("Unable to retrieve bucket lifecycle config: {0}".format(e))
```

*Key Values*
* `<number-of-days>` - Number of days to keep the restored file

*SDK References*
* [restore_object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}

### View HEAD information for an object
```python
def retrieve_head_object(bucket_name, item_name):
    print("Retrieving HEAD for item: {0} from bucket: {1}".format(item_name, bucket_name))

    try:
        response = cos_cli.head_object(
            Bucket=bucket_name, 
            Key=item_name)

        print(json.dumps(response, indent=4, sort_keys=True, default=str))
    except ClientError as be:
        print("CLIENT ERROR: {0}\n".format(be))
    except Exception as e:
        log_error("Unable to retrieve bucket lifecycle config: {0}".format(e))
```

*SDK References*
* [head_object](https://ibm.github.io/ibm-cos-sdk-python/reference/services/s3.html#client){:new_window}

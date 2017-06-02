---

copyright:
  years: 2014, 2017
lastupdated: "2017-02-23"

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}


# Getting started with {{site.data.keyword.objectstorageshort}} {: #getting-started-with-object-storage}


{{site.data.keyword.objectstoragefull}} provides unstructured cloud data storage. You can store and access your content, as well as interactively compose and connect to apps and services.
{: shortdesc}


1. Provision your service instance from the {{site.data.keyword.Bluemix_notm}} catalog. Configure your instance and click **Create**. If you initially choose the **Leave Unbound** option for the **App** field, you can bind the service instance to your {{site.data.keyword.Bluemix_notm}} app later.

2. In your service instance dashboard, create a container to start storing objects.

3. Add a file to your container by using the **Actions** menu.

4. To test access to your objects, click **Download** and review the file.

5. When you're ready, [bind the service](/docs/services/reqnsi.html#add_service) to an application.

# About {{site.data.keyword.objectstorageshort}}

{{site.data.keyword.objectstoragefull}} provides you a fully provisioned account, based on the Swift API, to manage your data.
{: shortdesc}


## How the service works

With {{site.data.keyword.objectstorageshort}}, your unstructured data is stored in a scalable, multi-tenant cloud environment. The service uses OpenStack Identity (Keystone) for authentication and can be accessed directly by using OpenStack Object Storage (Swift) API v1 calls. The service can be bound to a {{site.data.keyword.Bluemix_notm}} application or accessed from outside a {{site.data.keyword.Bluemix_notm}} application.

Provider side encryption is not provided. You can use the Key Protect service to encrypt your data. For more information, see [the service documentation](https://console.ng.bluemix.net/docs/services/keymgmt/index.html).

Both admins and developers can store and access objects as shown in the following diagram.


<dl>
  <dt> {{site.data.keyword.Bluemix_notm}} app </dt>
    <dd> You can bind the {{site.data.keyword.objectstorageshort}} service to a {{site.data.keyword.Bluemix_notm}} app. You must encrypt your data before uploading when using {{site.data.keyword.Bluemix_notm}} Public {{site.data.keyword.objectstorageshort}}. </dd>
  <dt> Client app </dt>
    <dd> You can access {{site.data.keyword.objectstorageshort}} directly from your application through a firewall on a private network. Provider side encryption is not provided. </dd>
  <dt> Keystone </dt>
    <dd> You authenticate your service with Keystone and receive an authorization token. </dd>
  <dt> OpenStack Swift API </dt>
    <dd> After you authenticate your instance, you can read and write to your stored objects by using the Swift API. </dd>
  <dt> Storage nodes </dt>
    <dd> The service maintains three copies of your data that it <a href="http://docs.openstack.org/developer/swift/overview_replication.html">replicates across multiple storage nodes</a>. </dd>
</dl>

![How {{site.data.keyword.objectstorageshort}} works as written above, shown in a diagram.](/images/OS_howitworks.png)
<caption> Figure 1. How {{site.data.keyword.Bluemix_notm}} Public {{site.data.keyword.objectstorageshort}} works </caption>


# Authenticating your {{site.data.keyword.objectstorageshort}} instance with Keystone

To interact with the service, you must authenticate your {{site.data.keyword.objectstorageshort}} instance with Keystone to obtain your URL and bearer token.
{: shortdesc}


Provisioning a new {{site.data.keyword.objectstorageshort}} instance creates an isolated Keystone project in the IBM Public Cloud. The Keystone credentials structure contains a complete set of attributes so that you can choose the OpenStack token request method or OpenStack SDK that best fits your app. When you bind a new app to the instance, a new Keystone user with access to the project is created. When you deprovision the instance, the project and user are deleted.

For more information about using OpenStack Swift and Keystone, view the <a href="http://docs.openstack.org" target="_blank">OpenStack documentation site. <img src="../../icons/launch-glyph.svg" alt="External link icon"></a>



1. Make a POST request to `https://identity.open.softlayer.com/v3/auth/tokens` as shown in the following cURL command.
  ```
  	curl -i \
  	  -H "Content-Type: application/json" \
  	  -d '
  	{
  		"auth": {
  			"identity": {
  				"methods": [
  					"password"
  				],
  				"password": {
  					"user": {
  						"id": "ad78b2a3f843466988afd077731c61fc",
  						"password": "XXXXXXXXXX"
  					}
  				}
  			},
  			"scope": {
  				"project": {
  					"id": "0f47b41b06d047f9aae3b33f1db061ed"
  				}
  			}
  		}
  	}' \
  	  https://identity.open.softlayer.com/v3/auth/tokens ; echo
  ```
  {: codeblock}

2. Select an `object-store` endpoint from the catalog response and take note. You need it to construct your full URL. The following response example is trimmed to show only information relevant to {{site.data.keyword.objectstorageshort}}.

  ```
  	HTTP/1.1 201 Created
  	Date: Mon, 29 Feb 2016 21:03:41 GMT
  	Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.1e-fips mod_wsgi/3.4 Python/2.7.5
  	X-Subject-Token: gAAAAABW1LIubUgqKl-eInzhZUHWEnXijp7t6_5inl4DTRLxDhNbJ25ly2X7bASNvH7ocxinaJu_kdhSfnHNRwPAeYY77Ii2Cwp02-bvxUA1S9lV_knT6EyCOW2mSBl_HuuDD2cEgdiKmyZTVt-RvDxhPKYD-rHkJz-dHO4Folg8TVXotilb1uw
  	Vary: X-Auth-Token
  	x-openstack-request-id: req-01e096c8-5393-4f98-8ff6-029c55e42524
  	Content-Length: 12051
  	Content-Type: application/json

  	{
  	  "token" : {
  	    "roles" : [
  	      {
  	        "id" : "f61f06a84f6443e880210fa986bd8691",
  	        "name" : "ObjectStorageOperator"
  	      }
  	    ],
  	    "catalog" : [
  	      {
  	        "endpoints" : [
  	          {
  	            "id" : "20cbfa6ff22b4a67a1484d30235bfc80",
  	            "region" : "london",
  	            "region_id" : "london",
  	            "url" : "https://lon.objectstorage.service.open.networklayer.com/v1/AUTH_3ecf7d7bac2c4eda89c03dd3afa7a0a3",
  	            "interface" : "admin"
  	          },
  	          {
  	            "id" : "38b8c081b11a452bb951698c334a406d",
  	            "region" : "london",
  	            "region_id" : "london",
  	            "url" : "https://lon.objectstorage.service.open.networklayer.com/v1/AUTH_3ecf7d7bac2c4eda89c03dd3afa7a0a3",
  	            "interface" : "internal"
  	          },
  	          {
  	            "id" : "4207049680fa4effbecd044c7448a8cb",
                "region" : "dallas",
                "region_id" : "dallas",
                "url" : "https://dal.objectstorage.open.softlayer.com/v1/AUTH_4abf7d7bac2c4eda89c03dd3afa7a0a3",
                "interface" : "public"
  	          },
  	          {
  	            "id" : "8a65a0cf38ac4211ad6a3c9c0eb337ff",
  	            "region" : "london",
  	            "region_id" : "london",
  	            "url" : "https://lon.objectstorage.open.softlayer.com/v1/AUTH_3ecf7d7bac2c4eda89c03dd3afa7a0a3",
  	            "interface" : "public"
  	          },
  	          {
  	            "id" : "a60cf32be624491d89170ef8264de5e8",
  	            "region" : "dallas",
  	            "region_id" : "dallas",
  	            "url" : "https://dal.objectstorage.service.open.networklayer.com/v1/AUTH_3ecf7d7bac2c4eda89c03dd3afa7a0a3",
  	            "interface" : "admin"
  	          },
  	          {
  	            "id" : "c769862200124a308d6748e418c971ba",
  	            "region" : "dallas",
  	            "region_id" : "dallas",
  	            "url" : "https://dal.objectstorage.service.open.networklayer.com/v1/AUTH_3ecf7d7bac2c4eda89c03dd3afa7a0a3",
  	            "interface" : "internal"
  	          }
  	        ],
  	        "id" : "896e4064cbe742afbf9a543c15f27ac0",
  	        "type" : "object-store",
  	        "name" : "swift"
  	      },
  	    ],
  	    "extras" : {

  	    },
  	    "user" : {
  	      "id" : "0b8aebd924ef4cc7aa9232f07e47e874",
  	      "name" : "user_87c094ce47a9feae3a137ffcbbfa098a888c12a8",
  	      "domain" : {
  	        "id" : "8753ff40ac1a4f4a9f162ad8026b6ce0",
  	        "name" : "757955"
  	      }
  	    },
  	    "expires_at" : "2016-02-29T22:03:42.061343Z",
  	    "audit_ids" : [
  	      "cbA-iL2dSheyB72PHd7q8Q"
  	    ],
  	    "issued_at" : "2016-02-29T21:03:42.000000Z",
  	    "project" : {
  	      "id" : "3ecf7d7bac2c4eda89c03dd3afa7a0a3",
  	      "name" : "object_storage_c1d8b3a1",
  	      "domain" : {
  	        "id" : "8753ff40ac1a4f4a9f162ad8026b6ce0",
  	        "name" : "757955"
  	      }
  	    },
  	    "methods" : [
  	      "password"
  	    ]
  	  }
  	}
  ```
  {: screen}

  <table>
  <caption> Table 1. Post request response explained </caption>
    <tr>
      <th> Response endpoint </th>
      <th> Explanation </th>
    </tr>
    <tr>
      <td> <code> X-Subject-Token </code> </td>
      <td> Your authentication token. </td>
    </tr>
    <tr>
      <td> <code> id </code> </td>
      <td> Your {{site.data.keyword.objectstorageshort}} instance ID. </td>
    </tr>
    <tr>
      <td> <code> region </code> </td>
      <td> Your storage region. Be sure to select the region that matches the field that is listed in your credentials. </td>
    </tr>
    <tr>
      <td> <code> url </code> </td>
      <td> Your {{site.data.keyword.objectstorageshort}} URL. </td>
    </tr>
    <tr>
      <td> <code> interface </code> </td>
      <td> The internal interface  is not accessible from {{site.data.keyword.Bluemix_notm}}. Use the public interface (<code>publicURL</code>). </td>
    </tr>
  </table>



# Generating service credentials {: #credentials}

Service credentials are used to provide authentication and security for the service. You can generate new credentials by using the CLI or the {{site.data.keyword.Bluemix_notm}} UI.
{: shortdesc}


## To add credentials in the UI:

1. In the **Service Credentials** tab on your service instance dashboard, click **New Credential** and give it a name.
2. Optional: In the **Add Inline Configuration Parameters** text field, input the credential information for the role with the [type of access](/docs/services/ObjectStorage/os_access_types.html) you want to give. The information must be formatted as a JSON payload.
    - To create an administrative user: `{"role":"admin"}`
    - To create a non-administrative user: `{"role":"member"}`
3. Click **Add**.


## To add credentials by using Cloud Foundry commands in the CLI:

1. If you are not logged in to {{site.data.keyword.Bluemix_notm}}, log in as a user with a developer role. You must be located within the space of the service instance you want to manage.
  ```
  cf login -a api.ng.bluemix.net -u <userid> -p <password> -o <organization> -s <space>
  ```
  {: pre}

2. Generate service credentials. Give your credential a name by replacing the variable
`service-key-name`. Optionally, you can also assign a [user role](/docs/services/ObjectStorage/os_access_types.html).

  ```
  cf create-service-key "<service_instance_name>" <service-key-name> -c '{"role":"<user_role>"}'
  ```
  {: pre}

  Example:
  ```
  cf create-service-key "Object-Storage-AclTest" GeorgeKey -c '{"role":"member"}'
  ```
  {: screen}

3. Validate the credentials for the key you created.

  ```
  cf service-keys <service_instance_name>
  ```
  {: pre}

  Example key for an instance that is named Object-Storage-Acl-Test for a user with a member role:

  ```
  {
    "auth_url": "https://identity.open.softlayer.com",
    "domainId": "8753ff40ac1a4f4a9f162ad8026b6ce0",
    "domainName": "757955",
    "password": "xxxxxxxx",
    "project": "object_storage_6046b6e2_ee53_4884_916f_89c65e4d408e",
    "projectId": "c727d7e248b448f6b268f31a1bd8691e",
    "region": "dallas",
    "role": "member",
    "userId": "3c5b516655e4479881d3d216895faedb",
    "username": "member_2afbeea1d58b1867f46c699553d1e4513e7df83a"
  }
  ```
  {: screen}



## To add credentials by using cURL commands in the CLI:

1. If you are not logged in to {{site.data.keyword.Bluemix_notm}}, log in as a user with a developer role. You must be located within the space of the service instance you want to manage.

  ```
  cf login -a api.ng.bluemix.net -u <userid> -p <password> -o <organization> -s <space>
  ```
  {: pre}

2. Run the following command to generate service credentials.

  ```
  curl "https://api.ng.bluemix.net/v2/service_keys" -d '{   "service_instance_guid": "<service_instance_guid>",   "name: <service_instance_name>", "role: <user_role>"}' -X POST -H "Authorization: <bearer_token>" -H "Content-Type: <content_type" -H "Cookie: <cookie>"
  ```
  {: pre}

  <table>
  <caption> Table 1. cURL service credential variables explained </caption>
    <tr>
      <th> Variable  </th>
      <th> Explanation </th>
    </tr>
    <tr>
      <td> ```https://api.ng.bluemix.net/v2/service_keys``` </td>
      <td> The service key endpoint.  </td>
    </tr>
    <tr>
      <td><i> service_instance_guid </i></td>
      <td> Can be found in your URL.  </td>
    </tr>
    <tr>
      <td><i> name </i></td>
      <td> The name of your service instance. </td>
    </tr>
    <tr>
      <td><i> role </i></td>
      <td> Specify the <a href= /docs/services/ObjectStorage/os_constructing.html>user's role</a> as either an admin or a member. </td>
    </tr>
    <tr>
      <td><i> bearer_token </i></td>
      <td> The token that you received when you authenticated your instance with Keystone. </td>
    </tr>
  </table>

3. Validate your credentials by running the following command.

  ```
  curl "https://api.ng.bluemix.net/v2/service_instances/b9656309-d994-4dec-a71f-8eac6e2fc7dc/service_keys" -X GET  -H "Authorization: <bearer_token>" -H "Cookie: "
  ```
  {: screen}



# Configuring the CLI to use Swift and Cloud Foundry commands

To interact with the {{site.data.keyword.objectstorageshort}} service by using Swift or Cloud Foundry commands, you must install the prerequisite software.
{: shortdesc}


## Installing the Swift client {: #install-swift-client}

If you haven't already, install the following prerequisite software.
* Python 2.7 or later
* setuptools package
* pip package
* Cloud Foundry CLI


To install the Python Swift client, run the following command:
```
pip install python-swiftclient
```
{: pre}

To install the Python Keystone client, run the following command:
```
pip install python-keystoneclient
```
{: pre}

For more information about the prerequisites, see <a href="http://docs.openstack.org/user-guide/common/cli_install_openstack_command_line_clients.html#install-the-prerequisite-software" target="_blank">the OpenStack documentation. <img src="../../icons/launch-glyph.svg" alt="External link icon"></a>



## Setting up the client {: #setup-swift-client}

To configure the Swift client, you must `export` your authentication information. You can [generate credentials](/docs/services/ObjectStorage/os_credentials.html) through the CLI by using Cloud Foundry or cURL commands or through the {{site.data.keyword.Bluemix_notm}} UI. The client takes the information from the environment variables in the following table.

<table>
<caption> Table 1. Environment variables and descriptions </caption>
  <tr>
    <th> Environment variable </th>
    <th> Description </th>
  </tr>
  <tr>
    <td> <code> OS_USER_ID </code> </td>
    <td> Your {{site.data.keyword.objectstorageshort}} user ID. </td>
  </tr>
  <tr>
    <td> <code> OS_PASSWORD </code> </td>
    <td> The password for your {{site.data.keyword.objectstorageshort}} instance. </td>
  </tr>
  <tr>
    <td> <code> OS_PROJECT_ID </code> </td>
    <td> Your project ID. </td>
  </tr>
  <tr>
    <td> <code> OS_REGION_NAME </code> </td>
    <td> The region in which your objects are stored. {{site.data.keyword.objectstorageshort}} is available in the Dallas and London regions. </td>
  </tr>
  <tr>
    <td> <code> OS_AUTH_URL </code> </td>
    <td> The endpoint URL. Be sure that <code>/v3</code> is added to the end of the URL. </td>
  </tr>
</table>



To export your authentication information, input your credentials and run the following command:
```
export OS_USER_ID=
export OS_PASSWORD=
export OS_PROJECT_ID=
export OS_AUTH_URL=
export OS_REGION_NAME=
export OS_IDENTITY_API_VERSION=
export OS_AUTH_VERSION=

swift auth
```
{: codeblock}


Example:
```
export OS_USER_ID=24a20b8e4e724f5fa9e7bfdc79ca7e85
export OS_PASSWORD=*******
export OS_PROJECT_ID=383ec90b22ff4ba4a78636f4e989d5b1
export OS_AUTH_URL=https://identity.open.softlayer.com/v3
export OS_REGION_NAME=dallas
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_VERSION=3

swift auth
```
{: screen}

# Constructing your {{site.data.keyword.objectstorageshort}} URL to use the Swift REST API

You can use the Swift REST API with a command-line client interface, such as cURL, or call the API from your application.
{: shortdesc}


For a comprehensive list of the {{site.data.keyword.objectstorageshort}} REST API options and examples, see the <a href="http://developer.openstack.org/api-ref-objectstorage-v1.html" target="_blank">OpenStack Swift API complete reference. <img src="../../icons/launch-glyph.svg" alt="External link icon"></a>



Before you can compose your URL, you must [authenticate](/docs/services/ObjectStorage/os_authenticate.html) your service instance with Keystone. Be sure to note your catalog response. It will look similar to the following example.

```
{
  "id" : "4207049680fa4effbecd044c7448a8cb",
  "region" : "dallas",
  "region_id" : "dallas",
  "url" : "https://dal.objectstorage.open.softlayer.com/v1/AUTH_4abf7d7bac2c4eda89c03dd3afa7a0a3",
  "interface" : "public"
},
```
{: codeblock}


Add the namespace of your container and object to the end of your {{site.data.keyword.objectstorageshort}} URL as shown in the following image.

![{{site.data.keyword.objectstorageshort}} URL pieces shown in an example image](images/Swift_URL.png)

Figure 1. {{site.data.keyword.objectstorageshort}} URL example


# Managing objects

You can use the Swift CLI, API, or Bluemix UI to manage your objects and containers.
{: shortdesc}

Before you can manage objects, be sure that you have [authenticated](/docs/services/ObjectStorage/os_authenticate.html) your service instance, [generated](/docs/services/ObjectStorage/os_credentials.html) service credentials, and [configured](/docs/services/ObjectStorage/os_configuring.html) the Swift CLI.

Keep the following points in mind when you are managing objects:
  * There is no limit to the amount of data that you can store, but each upload can be no more than 5 GB. If you need to upload files larger than 5 GB follow the steps that are found in [storing large objects](/docs/services/ObjectStorage/os_large_files.html).
  * Swift does not have a true directory structure, but you can add an [existing directory](/docs/services/ObjectStorage/os_directories.html) to your commands.
  * To avoid unintentionally overwriting a file, [set up object versioning](/docs/services/ObjectStorage/os_versioning.html).
  * You can [schedule a time](/docs/services/ObjectStorage/os_deletion.html) for your objects to be deleted.


# Storing objects

You can upload objects to storage by using the UI or CLI. Uploading objects is limited to a maximum size of 5 GB in a single upload. However, you can upload larger objects by segmenting them into smaller objects.
{: shortdesc}


## Storing objects in containers through the UI {: #storing-ui}

**Note**: When you upload a file with the same name, {{site.data.keyword.objectstorageshort}} replaces the stored file with the new file. If you download a file and make edits, be sure to give the file another name, or [set up object versioning](/docs/services/ObjectStorage/os_versioning.html) before you upload to keep both versions of the file.


1. From your service instance dashboard, add a container from the **Actions** drop-down menu.
2. Name your container and click **CREATE**.
3. From the **Actions** drop-down menu, select **Add Files**. A dialog box opens.
4. Select the file, or files you want to upload and click **Open**.


## Storing objects in containers through the CLI {: #storing-cli}

1. If you are not logged in to {{site.data.keyword.Bluemix_notm}}, log in to the org and space that contains your instance of {{site.data.keyword.objectstorageshort}}.

  ```
  cf login -a api.ng.bluemix.net -u <userid> -p <password> -o <organization> -s <space>
  ```
  {: pre}

2. Create an {{site.data.keyword.objectstorageshort}} container by running the following command. The *container_name* variable is set, by you, now.

  ```
  swift post <container_name>
  ```
  {: pre}

  **Note**: If you receive an error message, confirm that the [the prerequisite software](/docs/services/ObjectStorage/os_configuring.html#install-swift-client) is installed.

3. Optional: To verify that your container was created, run the following command to list your containers.

  ```
  swift list
  ```
  {: pre}

4. To prevent data destruction by unintentional over-writes, [set up object versioning](/docs/services/ObjectStorage/os_versioning.html). If you do not want object versioning, list your existing files in the store, and if necessary, rename the directory or the files before you upload.

5. Upload a file to your container by running the following command.

  ```
  swift upload <container_name> <file_name>
  ```
  {: pre}

  **Note**: To upload files larger than 5 GB [extra steps are needed](/docs/services/ObjectStorage/os_large_files.html).

6. Optional: To verify that upload was successful, view the content of your container by running the following command.

  ```
  swift list <container_name>
  ```
  {: pre}


# Storing large objects {: #large-files}

Uploads are limited to a maximum size of 5 GB for a single upload. However, you can segment larger objects into smaller pieces and use a manifest file to concatenate the segments. Once an object is concatenated, there is no maximum size.
{: shortdesc}

Large objects can either be dynamic or static. With static large objects (SLO), segments don't have to be in the same container; each segment can be stored in any container and given any name. With dynamic large objects, the Swift client, creates container and numbered segments are uploaded in parallel to the container.


## Dynamic large objects: {: #dynamic}

You can upload dynamic large objects in two ways:
  * Have the Swift client handle everything automatically
  * Use the Swift API to do it yourself

### Using the Swift client to handle dynamic large objects

The Swift client uses the `-segment-size` parameter to break down your object into smaller pieces. The client creates a new container with the name of the container you want to upload the files to and adds a suffix with the segment number (`<container_name>_segments`). Segments are uploaded in parallel. After all of the segments are uploaded, they are downloaded as one concatenated object to a manifest file with the original file name.

1. After you've logged in to {{site.data.keyword.Bluemix_notm}} and you're ready to upload, run the following command to segment your file.
    ```
    swift upload <container_name> <file_name> --segment-size <size_in_bytes>
    ```
    {: pre}

### Using the Swift API to handle Dynamic Large Objects

You can segment the objects so that they are 5 GB or less yourself, and then upload them through the Swift API.

**Note**: When uploading, all of the segments must be uploaded before the manifest file. If the object is downloaded before all of the segments are uploaded, the downloaded object concatenates inconsistently.

You can upload large files by completing the following steps.

1. Sort the segments by name in the order in which they need to concatenate to form the original object.
2. Upload your segments into one container that is separate from the container that holds the manifest file. Throttling for uploads starts after the 10th segment is uploaded, and increases the upload time considerably.  

    ```
    curl -i -X PUT --data-binary @segment1 -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_name>/<object_name>/000001
    curl -i -X PUT --data-binary @segment2 -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_name>/<object_name>/000002
    ```
    {: pre}

3. Upload an empty manifest file with the header `X-Object-Manifest` set to the corresponding `<container>/prefix>` value.

    ```
    curl -i -X PUT -H "X-Auth-Token: <token>" -H "X-Object-Manifest: <container_name>/<object_name>/" https://<object-storage_url>/<manifest_container_name>/<object_name>
    ```
    {: pre}

    **Note**: The manifest file must be empty. If not, the content of the file is considered as one of the segments and falls in the order of concatenation that is dictated by the sorted names.
4. Download the object. As a result, you receive the whole object. You can add or remove segments without having to update the manifest file. Segments with the correct prefix remain part of the object. Deleting the manifest does not delete the segments.

    ```
    curl -i -O -H "X-Auth-Token: <token>" https://<object-storage_url>/<manifest_container_name>/<object_name>
    ```
    {: pre}


## Static large objects {: #static}

Static large objects use segments and a manifest file, but you have more control. With SLO, segments don't have to be in the same container; each segment can be stored in any container and given any name. However, segments must be at least 1 MB. You are not required to set a header for the manifest file, although the header “X-Static-Large-Object” is automatically added and set to true after a correct manifest is uploaded.
{: shortdesc}

The manifest file is a JSON document that provides details of the segments and must be uploaded after all the segments have been uploaded. The data that is provided for each segment in the manifest is compared to the metadata of the actual segments. If something doesn’t match, the manifest isn't uploaded.

<table>
<caption> Table.1 JSON attributes in the manifest file </caption>
  <tr>
    <th> Attribute </th>
    <th> Description </th>
  </tr>
  <tr>
    <td> <i> path </i> </td>
    <td> The location and name of the segment. Specified as container_name/object_name. </td>
  </tr>
  <tr>
    <td> <i> etag </i> </td>
    <td> Provided by the PUT request when the object is uploaded. You can also find it by doing a HEAD to the object. </td>
  </tr>
  <tr>
    <td> <i> size_bytes </i> </td>
    <td> The size of the object in bytes. </td>
  </tr>
</table>



### To upload large files

1. Run the following command to upload the segments. Throttling for uploads starts after the 10th segment is uploaded, and increases the upload time considerably.  

    ```
    curl -i -X PUT --data-binary @segment1 -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_one>/<segment>
    curl -i -X PUT --data-binary @segment2 -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_two>/<segment>
    curl -i -X PUT --data-binary @segment3 -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_one>/<segment>
    ```
    {: pre}

2. Build the manifest:

    ```
    [
        {
            "path": "<container_one>/<segment>",
            "etag": "e0ed3b751eb8d4b2c648d2dd78576e36",
            "size_bytes": 801882690
        },
        {
            "path": "container_two/<segment>",
            "etag": "65a301e71c82cbd325a5efe5877e1a24",
            "size_bytes": 1048576
        },
        {
            "path": "<container_one>/<segment>",
            "etag": "aea8b7462d527ad5ed0cfc22ea161062",
            "size_bytes": 138257
        }
    ]
    ```
    {: pre}

3. Upload the manifest file by adding the query `multipart-manifest=put` to the name of the manifest.

    ```
    curl -i -X PUT --data-binary @object_name -H "X-Auth-Token: <token>" https://<object-storage_url>/container_two/<object_name>?multipart-manifest=put
    ```
    {: pre}

4. Download the object.

    ```
    curl -O -X GET -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_two>/<object_name>
    ```
    {: pre}


## Working with static large objects

You can manage your files by using the following commands.

**Note**: To add or remove segments to the object, upload a new manifest file with a new list of segments. The manifest name can stay the same.

* To download the content of the manifest file, you must add the query `multipart-manifest=get` to your command. The content that you receive is not identical to the content that you uploaded.

    ```
    curl -O -X GET -H "X-Auth-Token:<token>" https://<object-storage_url>/<container_two>/<object_name>?multipart-manifest=get
    ```
    {: pre}

* To delete the manifest run the following command:

    ```
    curl -i -X DELETE -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_two>/<object_name>
    ```
    {: pre}

* To delete the manifest and all the segments, add the query `multipart-manifest=delete` after the name of the manifest:

    ```
    curl -i -X DELETE -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_two>/<object_name>?multipart-manifest=delete
    ```
    {: pre}




# Downloading objects

You can download your stored objects for review or editing through the UI or the CLI.
{: shortdesc}


## Downloading objects from the UI {: #downloading-ui}

1. To prevent data destruction by unintentional over-writes, [set up object versioning](/docs/services/ObjectStorage/os_versioning.html). If you don't want object versioning, rename the directory or the files before you download, if necessary.
2. In your service instance dashboard, select the container with the file you want to download.
3. Select the file.
4. In the **Actions** drop-down menu, select **Download File**.


## Downloading objects with the Swift CLI {: #downloading-cli}

1.  If you are not logged in to {{site.data.keyword.Bluemix_notm}}, log in to the org and space that contains your instance of {{site.data.keyword.objectstorageshort}}.

    ```
    cf login -a api.ng.bluemix.net -u <userid> -p <password> -o <organization> -s <space>
    ```
    {: pre}

2. To prevent data destruction by unintentional over-writes, [set up object versioning](/docs/services/ObjectStorage/os_versioning.html). If you do not want object versioning, list your existing files in the store, and if necessary, rename the directory or the files before you download.

3. Download a file by running the following command:

    ```
    swift download <container_name> <file_name>
    ```
    {: pre}



# Deleting objects

After you no longer have a need for them, you can delete objects and containers from your storage instance. You can manually do the delete, or you can [schedule a time](/docs/services/ObjectStorage/os_deletion.html#schedule-object-deletion) for the objects to expire.
{: shortdesc}

**Note**: If you delete your container, any objects that are stored within that container are deleted.


## Deleting objects and containers through the UI {: #deleting-ui}

1. In your service instance dashboard, select the container with the file that you no longer need.
2. Select **Delete File** from the **Actions** drop-down menu.
3. If you no longer have use for your container, select **Delete Container** from the **Actions** drop-down menu.



## Deleting objects and containers through the CLI {: #deleting-cli}

1.  If you are not logged in to {{site.data.keyword.Bluemix_notm}}, log in to the org and space that contains your instance of {{site.data.keyword.objectstorageshort}}.
  ```
  cf login -a api.ng.bluemix.net -u <userid> -p <password> -o <organization> -s <space>
  ```
  {: pre}

2. Optional: Confirm that you have a backup of your objects before you delete your files and containers.

3. Run the following command to delete a file:
  ```
  swift delete <container_name> <file_name>
  ```
  {: pre}

4. To delete your container, run the following command:
  ```
  swift delete <container_name>
  ```
  {: pre}



# Scheduling object deletion {: #schedule-object-deletion}


You can schedule the deletion of your objects by using either of the `X-Delete-At` or `X-Delete-After` headers.
{: shortdesc}

The `X-Delete-At` header takes an integer that represents the epoch time at which to delete the object. The `X-Delete_After` header takes an integer that represents the number of seconds after which the object is deleted.

**Note:** The actual deletion of an object might not happen at the exact time indicated. However, the object will expire at the specified time. At that time, the object is longer reachable. The actual deletion will take place the next time the swift-object-expirer daemon that is configured in your Swift cluster runs.

## To use Swift commands:

* To set the object to be deleted at a specific date and time, run the following command:

    ```
    swift post -H "X-Delete-At:<epoch_time>" <container_name> <object_name>
    ```
    {: pre}

    Example:
    To set the object to be deleted on "2016/04/01 08:00:00", you would run the following command:

    ```
    swift post -H "X-Delete-At:1459515600" container1 file7
    ```
    {: screen}

* To set the object to be deleted after a specific amount of time, use the following command:

    ```
    swift post -H "X-Delete-After:<number_of_seconds>" <container_name> <object_name>
    ```
    {: pre}

    Example:
    To set the object to be deleted an hour from now, you would run the following command:

    ```
    swift post -H "X-Delete-After:3600" container1 file7
    ```
    {: screen}

* To remove the expiration time from your object, use the following command:

    ```
    swift post -H "X-Remove-Delete-After:<number_of_seconds>" <container_name> <object_name>
    ```
    {: pre}



## To use cURL commands:

* To set the object to be deleted on "2016/04/01 08:00:00", use the following command:

    ```
    cURL -X POST -H "X-Auth-Token: <token>" -H "X-Delete-At:<epoch_time>" https://<object-storage_url>/<container_name>/<object_name>
    ```
    {: pre}

* To set the object to be deleted an hour from now, use the following command:

    ```
    cURL -X POST -H "X-Auth-Token: <token>" -H "X-Delete-After:<number_of_seconds>" https://<object-storage_url>/<container_name>/<object_name>
    ```
    {: pre}

* To check if the object has the header, use the following command:

    ```
    cURL -I -H "X-Auth-Token: <token>" https://<object-storage_url>/<container_name>/<object_name>
    ```
    {: pre}

* To remove the expiration time, use the following command:

    ```
    cURL -X POST -H "X-Auth-Token: <token>" -H "X-Remove-Delete-At:<epoch_time>" https://<object-storage_url>/<container_name>/<object_name>
    ```
    {: pre}


# Managing access

Access Control Lists can be used to manage access to the service. [Admin users](/docs/services/ObjectStorage/os_access_types.html) can grant read and write access, as well as remove access.
{: shortdesc}

Before managing access by using the UI or the Swift CLI, ensure that you have configured the service and begun storing objects.

When you are working with access control lists, keep the following points in mind:
  * When a user creates a container, they are automatically granted admin privileges to that container.
  * Access control lists are not available for the service instance, storage account, or at the project level. Access can be managed at the container or object level only.
  * Be sure to verify that access to a container was [removed](/docs/services/ObjectStorage/os_remove_access.html).


# Types of access

{{site.data.keyword.objectstorageshort}} users can be either administrative or non-administrative. Access control lists are enabled by administrative users at the container level.
{: shortdesc}

<table>
<caption> Table 1. User roles defined </caption>
  <tr>
    <th> Administrative users (admin) </th>
    <th> Non-administrative users (member) </th>
  </tr>
  <tr>
    <td> Manage access control </td>
    <td> By default, have no access to the service or its containers </td>
  </tr>
  <tr>
    <td> Can create and delete containers </td>
    <td> Can perform actions based on the containers read/write ACLs </td>
  </tr>
  <tr>
    <td> Can read and write to containers </td>
    <td> Can perform actions as determined by the admin </td>
  </tr>
</table>


You can manage {{site.data.keyword.objectstorageshort}} users through the {{site.data.keyword.Bluemix_notm}} user interface, the Cloud Foundry API, or the Cloud Foundry CLI.



# Granting read access

An {{site.data.keyword.objectstorageshort}} user with an [admin role](/docs/services/ObjectStorage/os_access_types.html) can grant read access to a container for another user and manipulate read ACL combinations.
{: shortdesc}

<table>
<caption> Table 1. Read access permissions by option </caption>
  <tr>
    <th> Permission </th>
    <th> Read ACL options </th>
  </tr>
  <tr>
    <td> Read for all referers regardless of account affiliation </td>
    <td> <code> .r,&#42;  </code> </td>
  </tr>
  <tr>
    <td> Read and list for all referers and listing </td>
    <td> <code> .r:&#42;,.rlistings </code> </td>
  </tr>
  <tr>
    <td> Read and list for a specified user in a specific project </td>
    <td> <code> project_id:user_id </code> </td>
  </tr>
  <tr>
    <td> Read and list for a specified user in every project </td>
    <td> <code> &#42;:user_id </code> </td>
  </tr>
  <tr>
    <td> Read and list for every user in a specified project </td>
    <td> <code> project_id:&#42; </code> </td>
  </tr>
  <tr>
    <td> Read and list for every user in every project  </td>
    <td> <code> &#42;:&#42; </code> </td>
  </tr>
</table>


1. Authenticate your credentials. You can use either the credentials that are found in the service credentials tab of the UI or you can generate new credentials. For more information about generating new credentials, see [Generating service credentials](/docs/services/ObjectStorage/os_credentials.html). You receive your {{site.data.keyword.objectstorageshort}} URL and authentication token as an output.

    Swift command:

    ```
    export OS_USER_ID=<user_id>
    export OS_PASSWORD=<password>
    export OS_TENANT_ID=<project_id>
    export OS_AUTH_URL=https://identity.open.softlayer.com/v3
    export OS_REGION_NAME=<region>
    export OS_IDENTITY_API_VERSION=3
    export OS_AUTH_VERSION=3

    swift auth
    ```
    {: codeblock}

    cURL command:

    ```
    curl -i -H "X-Auth-User: <user_id>" -H "X-Auth-Key: <password>" <auth_url>
    ```
    {: pre}

2. Grant read access by running the following command:
    Swift command:

    ```
    swift post <container_name> --read-acl "<user_id>:<project_id>"
    ```
    {: pre}

    cURL command:

    ```
    curl -i <OS_STORAGE_URL> -X POST -H "Content-Length: 0" -H "X-Container-Read: <tenant_id>:<project_id>" -H "X-Auth-Token: <OS_AUTH_TOKEN>"
    ```
    {: pre}
    **Note**: Use a comma (,) to separate access control lists.

3. Verify the Read ACL value.

    Swift command:

    ```
    swift stat <container_name>
    ```
    {: pre}

    cURL command:

    ```
    curl -i <OS_STORAGE_URL> -I -H "X-Auth-Token:<OS_AUTH_TOKEN>"
    ```
    {: pre}

    Example: The `X-Container-Read` value shows to which container and to whom read access is granted.

    ```
    HTTP/1.1 204 No Content
    Content-Length: 0
    X-Container-Object-Count: 1
    Accept-Ranges: bytes
    X-Storage-Policy: standard
    X-Container-Read: c727d7e248b448f6b268f31a1bd8691e:3c5b516655e4479881d3d216895faedb
    X-Container-Bytes-Used: 31512
    X-Timestamp: 1462818314.11220
    Content-Type: text/plain; charset=utf-8
    X-Trans-Id: txad7fe001da274b9ba39a6-005772e4d6
    Date: Tue, 28 Jun 2016 20:57:58 GMT
    ```
    {: screen}


# Granting write access

An {{site.data.keyword.objectstorageshort}} user with an [admin role](/docs/services/ObjectStorage/os_access_types.html) can grant write access for another user and manipulate write ACL combinations.
{: shortdesc}

<table>
<caption> Table 1. Write access permissions by option </caption>
  <tr>
    <th> Permission </th>
    <th> Write ACL options </th>
  </tr>
  <tr>
    <td> Write for a specified user in a specific project </td>
    <td> <code> project_id:user_id </code> </td>
  </tr>
  <tr>
    <td> Write for a specified user in every project </td>
    <td> <code> &#42;:user_id </code> </td>
  </tr>
  <tr>
    <td> Write for every user in a specified project </td>
    <td>  <code> project_id:&#42; </code> </td>
  </tr>
  <tr>
    <td> Write for every user in every project </td>
    <td>  <code> &#42;:&#42; </code> </td>
  </tr>
</table>



1. Authenticate your credentials by using the information in the service credentials you created.  You receive your {{site.data.keyword.objectstorageshort}} URL and authentication token as an output.

    Swift command:

    ```
    export OS_USER_ID="<user_id>"
    export OS_PASSWORD="<password>"
    export OS_TENANT_ID=<project_id>
    export OS_AUTH_URL=https://identity.open.softlayer.com/v3
    export OS_REGION_NAME=<region>
    export OS_IDENTITY_API_VERSION=3
    export OS_AUTH_VERSION=3

    swift auth
    ```
    {: codeblock}

    cURL command:

    ```
    curl -i -H "X-Auth-User:< user_id>" -H "X-Auth-Key:< password>" https://identity.open.softlayer.com/v3
    ```
    {: pre}

2. Grant write access by running the following command.

    Swift command:

    ```
    swift post <container_name> --write-acl "<user_id>:<project_id>"
    ```
    {: pre}

    cURL command:

    ```
    curl -i <OS_STORAGE_URL> -X POST -H "Content-Length: 0" -H "X-Container-Write: <user_id>: <project_id>" -H "X-Auth-Token:<OS_AUTH_TOKEN>"
    ```
    {: pre}

    **Note**: Use a comma (,) to separate access control lists.

3. Verify the write ACL value.

    Swift command:

    ```
    swift stat <container_name>
    ```
    {: pre}

    cURL command:

    ```
    curl -i <OS_STORAGE_URL> -I -H "X-Auth-Token:<OS_AUTH_TOKEN>"
    ```
    {: pre}


# Removing access

You can remove access to a container or object by using access control lists.
{: shortdesc}

To remove read ACLs from a container, run one of the following commands.

* Swift command:

```
swift post <container_name> --read-acl “”
```
{: pre}

* cURL command:

```
curl -i <OS_STORAGE_URL> -X POST -H "Content-Length: 0" -H "X-Container-Read: " -H "X-Auth-Token: <OS_AUTH_TOKEN>"
```
{: pre}

To remove write ACLs from a container, run one of the following commands.

* Swift command:

```
swift post <container_name> --write-acl “”
```
{: pre}

* cURL command:

```
curl -i <OS_STORAGE_URL> -X POST -H "Content-Length: 0" -H "X-Container-Write: " -H "X-Auth-Token: <OS_AUTH_TOKEN>"
```
{: pre}

To verify that you have removed an ACL run one of the following commands.

* Swift command:

```
swift stat <container_name>
```
{: pre}

* The following example output shows both the Read ACL and Write ACL as blank, which means that access has been removed.

```
         Account: AUTH_c727d7e248b448f6b268f31a1bd8691e
       Container: Test
         Objects: 1
           Bytes: 31512
        Read ACL:
       Write ACL:
         Sync To:
        Sync Key:
   Accept-Ranges: bytes
X-Storage-Policy: standard
     X-Timestamp: 1462818314.11220
      X-Trans-Id: txf04968bc9ef8431982b77-005772e34b
    Content-Type: text/plain; charset=utf-8
```
{: screen}

* cURL command:

```
curl -i <OS_STORAGE_URL> -I -H "X-Auth-Token: <OS_AUTH_TOKEN>"
```
{: pre}


# Unbinding and deprovisioning your  {{site.data.keyword.objectstorageshort}} instance {: #deprovisioning-object-storage}

If the {{site.data.keyword.objectstorageshort}} service is bound to your Cloud Foundry App, but you no longer need storage, you can unbind and deprovision your instance.
{: shortdesc}


## Unbinding your instance

You can maintain your saved data and unbind the service from your Cloud Foundry app. The {{site.data.keyword.objectstorageshort}} account is not deleted until the service is deprovisioned.

**Attention**: If you unbind an {{site.data.keyword.objectstorageshort}} instance from a {{site.data.keyword.Bluemix_notm}} application, or delete the service key, all of your credentials for that instance are deleted, and cannot be restored. You can generate new cloud credentials by rebinding your instance, or by creating a new service key.

1. To see the services that are bound to your app, click the connections tab in your Cloud Foundry application.
2. Locate the service you that you want to unbind and click the menu button on the service tile.
3. Select **Unbind service**.
4. Clear the **Delete service instance** box and click **OK**.
5. Click **Restage** for your change to take effect.



## Deprovisioning your instance

If you have an instance of {{site.data.keyword.objectstorageshort}} that you no longer need, you can delete the instance.

**Attention**: When you deprovision an {{site.data.keyword.objectstorageshort}}  instance, the cloud project and Swift account are deleted. All containers and objects in the deprovisioned instance are deleted, and cannot be restored.

1. To see the services that are bound to your app, click the connections tab in your Cloud Foundry application.
2. Locate the service you that you want to deprovision and click the menu button on the service tile.
3. Select **Delete service**.
4. Click **Delete** to confirm.
5. Click **Restage** for your change to take effect.


# FAQ {: #faq}

This FAQ provides answers to common questions about the {{site.data.keyword.objectstorageshort}} service.
{: shortdesc}


## What accounts and payment plans can I use for {{site.data.keyword.objectstorageshort}}? {: #account-payment}

The {{site.data.keyword.objectstorageshort}} service offers two plan options, Free and Standard. [Pricing](https://www.ibm.com/cloud-computing/bluemix/pricing/) varies depending on the chosen plan.

<table>
<caption> Table 1. Comparison of Free and Standard plans </caption>
  <tr>
    <th> Free plan </th>
    <th> Standard plan </th>
  </tr>
  <tr>
    <td> Allows only one service instance to exist in a {{site.data.keyword.Bluemix_notm}} organization at a time </td>
    <td> Unlimited service instances </td>
  </tr>
  <tr>
    <td> Available to everyone </td>
    <td> Available only to {{site.data.keyword.Bluemix_notm}} paid accounts and IBM internal users </td>
  </tr>
  <tr>
    <td> Free </td>
    <td> Pay-as-you-go or subscription payment plans </td>
  </tr>
  <tr>
    <td> Includes an introductory 5 GB storage usage limit </td>
    <td> Unlimited storage </td>
  </tr>
</table>

**Attention**: Users working with a {{site.data.keyword.Bluemix_notm}} trial account are able to use the Free plan. After the time in your trial expires, the associated {{site.data.keyword.objectstorageshort}} service instance will be disabled, meaning that you will be unable to access the storage account. After 30 days, your {{site.data.keyword.Bluemix_notm}} account will be purged and all data deleted. To avoid data loss, upgrade to a [Paid {{site.data.keyword.Bluemix_notm}} account](/docs/admin/account.html) as soon as possible.

## How do I change my plan? {: #changeplan}  
Instances that are created through the Beta or on the Free plan can be upgraded to the Standard plan. The associated organization must be a {{site.data.keyword.Bluemix_notm}} paid account. Trial accounts with Object Storage instances cannot be upgraded to the Standard plan, and instances on the Standard plan cannot be downgraded to other plans. When you upgrade, your service instance and customer data are moved to the new plan.

To upgrade:
1.	In the {{site.data.keyword.objectstorageshort}} user interface, click **Plan**.
2.	Select **Standard** as the new plan and then click **Save**.

You can also [change your payment plan](/docs/pricing/index.html#changing) by using the command line interface.

## How will I be charged and billed for my use of {{site.data.keyword.objectstorageshort}}? {: #charge-bill}

The {{site.data.keyword.objectstorageshort}} service charges only for what you use.  There are no fees or commitments to begin using the service. You are not charged for API requests or inbound data network traffic.

{{site.data.keyword.objectstorageshort}} is billed based on your storage usage within the billing cycle. This includes all object data in containers that you created within your {{site.data.keyword.Bluemix_notm}} organization.

An Outbound Data Transfer charge applies whenever data is read from any of your object containers over the public network. All bandwidth that is consumed in the billing cycle is billed as Public Outbound Bandwidth.

At the end of the billing cycle, {{site.data.keyword.Bluemix_notm}} automatically bills you for usage during that billing period. You can view your charges for the current billing period via {{site.data.keyword.Bluemix_notm}} reporting.

## Are my {{site.data.keyword.Bluemix_notm}} region and my storage region the same thing? {: #region}

The {{site.data.keyword.objectstorageshort}} service supports the Dallas and London storage regions. These storage regions are independent of the {{site.data.keyword.Bluemix_notm}} region, such as US-South and United Kingdom, in which the {{site.data.keyword.objectstorageshort}} service instance is created. If you create an instance in the US-South {{site.data.keyword.Bluemix_notm}} region, you can read and write data to either the Dallas or London storage region. The storage region defaultsbased on your {{site.data.keyword.Bluemix_notm}} region. You can switch regions by selecting another region in from the drop-down list in the UI.



# {{site.data.keyword.objectstorageshort}} troubleshooting
{: #troubleshooting}


Here are the answers to common troubleshooting questions about using {{site.data.keyword.objectstoragefull}}.
{: shortdesc}

## Unrecognized token contentpack returned when using openstack4J with Liberty Profile
{: #unrecognized_token}


The following stacktrace might occur when using openstack4j with the Liberty Profile:
```
Exception thrown by application class 'org.openstack4j.connectors.okhttp.HttpResponseImpl.readEntity:124'
org.openstack4j.api.exceptions.ClientResponseException: Unrecognized token 'contentpack': was expecting ('true', 'false' or 'null') at [Source: contentpack ; line: 1, column: 12]
at org.openstack4j.connectors.okhttp.HttpResponseImpl.readEntity(HttpResponseImpl.java:124)
at org.openstack4j.core.transport.HttpEntityHandler.handle(HttpEntityHandler.java:56)
at org.openstack4j.connectors.okhttp.HttpResponseImpl.getEntity(HttpResponseImpl.java:68)
at org.openstack4j.openstack.internal.BaseOpenStackService$Invocation.execute(BaseOpenStackService.java:169)
at org.openstack4j.openstack.internal.BaseOpenStackService$Invocation.execute(BaseOpenStackService.java:163)
at org.openstack4j.openstack.storage.object.internal.ObjectStorageContainerServiceImpl.list(ObjectStorageContainerServiceImpl.java:41)
at com.mimotic.SecureMessageApp.HelloResource.getInformation(HelloResource.java:47)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
at java.lang.reflect.Method.invoke(Unknown Source)
```
{: screen}
{: tsSymptoms}


This problem results from a class-loading issue, where the openstack4j library contains some of the same packages that are provided in the Liberty profile.  For example, OpenStack4j uses JERSEY, which might collide with the Wink libs.
{: tsCauses}


You can fix this problem by either:
{: tsResolve}
  * Using reverse classloading (parentLast).
  * Excluding jaxrs from the enabled features.


## Getting help and support for {{site.data.keyword.objectstorageshort}}
{: #gettinghelp}

If you have problems or questions when using {{site.data.keyword.objectstoragefull}}, you can get help by searching for information or by asking questions through a forum. You can also open a support ticket.

When using the forums to ask a question, tag your question so that it is seen by the {{site.data.keyword.Bluemix_notm}} development teams.

* If you have technical questions about {{site.data.keyword.objectstorageshort}}, post your question on <a href="http://stackoverflow.com/search?q=object-storage+ibm-bluemix" target="_blank">Stack Overflow <img src="../../icons/launch-glyph.svg" alt="External link icon"></a> and tag your question with "ibm-bluemix" and "object-storage".
* For questions about the service and getting started instructions, use the <a href="https://developer.ibm.com/answers/topics/objectstorage/?smartspace=bluemix" target="_blank">IBM developerWorks dW Answers <img src="../../icons/launch-glyph.svg" alt="External link icon"></a> forum. Include the  "objectstorage" and "bluemix" tags.

See [Getting help](/docs/support/index.html#getting-help) for more details about using the forums.

For information about opening an IBM support ticket, or about support levels and ticket severities, see [Contacting support](/docs/support/index.html#contacting-support).

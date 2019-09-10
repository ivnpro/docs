---
editable: false
---

# Method create
Creates an image in the specified folder.
 
You can create an image from a disk, snapshot, other image or URI.
Method starts an asynchronous operation that can be cancelled while it is in progress.
 
## HTTP request {#https-request}
```
POST https://compute.api.cloud.yandex.net/compute/v1/images
```
 
## Body parameters {#body_params}
 
```json 
{
  "folderId": "string",
  "name": "string",
  "description": "string",
  "labels": "object",
  "family": "string",
  "minDiskSize": "string",
  "productIds": [
    "string"
  ],
  "os": {
    "type": "string"
  },

  //  includes only one of the fields `imageId`, `diskId`, `snapshotId`, `uri`
  "imageId": "string",
  "diskId": "string",
  "snapshotId": "string",
  "uri": "string",
  // end of the list of possible fields

}
```

 
Field | Description
--- | ---
folderId | **string**<br><p>Required. ID of the folder to create an image in. To get the folder ID, use a <a href="/docs/resource-manager/api-ref/Folder/list">list</a> request.</p> <p>The maximum string length in characters is 50.</p> 
name | **string**<br><p>Name of the image.</p> <p>Value must match the regular expression <code>\|[a-z][-a-z0-9]{1,61}[a-z0-9]</code>.</p> 
description | **string**<br><p>Description of the image.</p> <p>The maximum string length in characters is 256.</p> 
labels | **object**<br><p>Resource labels as <code>key:value</code> pairs.</p> <p>No more than 64 per resource. The string length in characters for each key must be 1-63. Each key must match the regular expression <code>[a-z][-_0-9a-z]*</code>. The maximum string length in characters for each value is 63. Each value must match the regular expression <code>[-_0-9a-z]*</code>.</p> 
family | **string**<br><p>The name of the image family to which this image belongs. For more information, see <a href="/docs/compute/concepts/image#family">Image family</a>.</p> <p>To get an information about the most recent image from a family, use a <a href="/docs/compute/api-ref/Image/getLatestByFamily">getLatestByFamily</a> request.</p> <p>Value must match the regular expression <code>\|[a-z][-a-z0-9]{1,61}[a-z0-9]</code>.</p> 
minDiskSize | **string** (int64)<br><p>Minimum size of the disk that will be created from this image. Specified in bytes. Should be more than the volume of source data.</p> <p>Acceptable values are 4194304 to 4398046511104, inclusive.</p> 
productIds[] | **string**<br><p>License IDs that indicate which licenses are attached to this resource. License IDs are used to calculate additional charges for the use of the virtual machine.</p> <p>The correct license ID is generated by Yandex.Cloud. IDs are inherited by new resources created from this resource.</p> <p>If you know the license IDs, specify them when you create the image. For example, if you create a disk image using a third-party utility and load it into Yandex Object Storage, the license IDs will be lost. You can specify them in this request.</p> <p>The maximum string length in characters for each value is 50.</p> 
os | **object**<br>Operating system that is contained in the image.  If not specified and you used the `image_id` or `disk_id` field to set the source, then the value can be inherited from the source resource.<br>
os.<br>type | **string**<br><p>Operating system type. The default is <code>LINUX</code>.</p> <p>This field is used to correctly emulate a vCPU and calculate the cost of using an instance.</p> <ul> <li>LINUX: Linux operating system.</li> <li>WINDOWS: Windows operating system.</li> </ul> 
imageId | **string** <br> includes only one of the fields `imageId`, `diskId`, `snapshotId`, `uri`<br><br><p>ID of the source image to create the new image from.</p> <p>The maximum string length in characters is 50.</p> 
diskId | **string** <br> includes only one of the fields `imageId`, `diskId`, `snapshotId`, `uri`<br><br><p>ID of the disk to create the image from.</p> <p>The maximum string length in characters is 50.</p> 
snapshotId | **string** <br> includes only one of the fields `imageId`, `diskId`, `snapshotId`, `uri`<br><br><p>ID of the snapshot to create the image from.</p> <p>The maximum string length in characters is 50.</p> 
uri | **string** <br> includes only one of the fields `imageId`, `diskId`, `snapshotId`, `uri`<br><br><p>URI of the source image to create the new image from. Currently only supports links to images that are stored in Yandex Object Storage. Currently only supports Qcow2, VMDK, and VHD formats.</p> 
 
## Response {#responses}
**HTTP Code: 200 - OK**

```json 
{
  "id": "string",
  "description": "string",
  "createdAt": "string",
  "createdBy": "string",
  "modifiedAt": "string",
  "done": true,
  "metadata": "object",

  //  includes only one of the fields `error`, `response`
  "error": {
    "code": "integer",
    "message": "string",
    "details": [
      "object"
    ]
  },
  "response": "object",
  // end of the list of possible fields

}
```
An Operation resource. For more information, see [Operation](/docs/api-design-guide/concepts/operation).
 
Field | Description
--- | ---
id | **string**<br><p>ID of the operation.</p> 
description | **string**<br><p>Description of the operation. 0-256 characters long.</p> 
createdAt | **string** (date-time)<br><p>Creation timestamp.</p> <p>String in <a href="https://www.ietf.org/rfc/rfc3339.txt">RFC3339</a> text format.</p> 
createdBy | **string**<br><p>ID of the user or service account who initiated the operation.</p> 
modifiedAt | **string** (date-time)<br><p>The time when the Operation resource was last modified.</p> <p>String in <a href="https://www.ietf.org/rfc/rfc3339.txt">RFC3339</a> text format.</p> 
done | **boolean** (boolean)<br><p>If the value is <code>false</code>, it means the operation is still in progress. If <code>true</code>, the operation is completed, and either <code>error</code> or <code>response</code> is available.</p> 
metadata | **object**<br><p>Service-specific metadata associated with the operation. It typically contains the ID of the target resource that the operation is performed on. Any method that returns a long-running operation should document the metadata type, if any.</p> 
error | **object**<br>The error result of the operation in case of failure or cancellation. <br> includes only one of the fields `error`, `response`<br><br><p>The error result of the operation in case of failure or cancellation.</p> 
error.<br>code | **integer** (int32)<br><p>Error code. An enum value of <a href="https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto">google.rpc.Code</a>.</p> 
error.<br>message | **string**<br><p>An error message.</p> 
error.<br>details[] | **object**<br><p>A list of messages that carry the error details.</p> 
response | **object** <br> includes only one of the fields `error`, `response`<br><br><p>The normal response of the operation in case of success. If the original method returns no data on success, such as Delete, the response is <a href="https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#empty">google.protobuf.Empty</a>. If the original method is the standard Create/Update, the response should be the target resource of the operation. Any method that returns a long-running operation should document the response type, if any.</p> 
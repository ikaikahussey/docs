---
title: Uploading Media
excerpt: Accept file uploads, and store your file-based content with your text-based content
order: 9
---

Your API can easily be configured to accept file uploads, allowing you to store file-based content along with your text-based content. All it takes is adding a [collection specification](../collections) with an additional `type` property to indicate that it is a `media` collection.

A media collection is similar to a standard collection but includes additional endpoints to allow uploading binary data. Along with the querying functionality available to standard collections, media collections respond to endpoints for generating signed URLs and uploading files.

## Media Collection Specification

To indicate that a collection is a media collection, add a `type` property to the specification file's `settings` block:

```json
{
  "settings": {
    "type": "media"
  }
}
```

For details regarding recommended fields to store in a media collection, see [collections](./collections).

## Configuration

There are two types of configuration for media collections in API - collection-level and global.

The only configuration option handled at collection-level is `signUploads`. Setting this to `true` will require the use of a signed token when uploading. See the [Signed URLs](#signed-urls) section below.

There is a default global configuration for media uploads. To override the global configuration, add a `media` block to the [main configuration file](getting-started/configuration):

```json
"media": {
  "tokenSecret": "difficult-wonderland-sunshine",
  "tokenExpiresIn": "1h",
  "basePath": "workspace/media",
  "pathFormat": "date",
  "storage": "disk"
}
```

|Property|Description|Default
|:--|:--|:---
|tokenSecret|The secret key used to sign and verify tokens when uploading media|`"catboat-beatific-drizzle"`
|tokenExpiresIn|The duration a signed token is valid for. Expressed in seconds or a string describing a time span (https://github.com/zeit/ms). Eg: `60`, `"2 days"`, `"10h"`, `"7d"`|`"1h"`
|basePath| When `"disk"` storage is used, `basePath` is either an absolute path or a path  relative to the directory where the application is run. When `"s3"` storage is used, `basePath` is a directory relative to the S3 bucket root.  | `"workspace/media"`
|pathFormat| Determines the format for the generation of subdirectories to store uploads. Possible values: `"none"`, `"date"`, `"datetime"`, `"sha1/4"`, `"sha1/5"`, `"sha1/8"` | `"date"`|
|storage| The storage handler to use. Determines where file uploads are stored. Possible values: `"disk"`, `"s3"`| `"disk"`

### Available path formats

The `pathFormat` property determines the directory structure that API will use when storing files. This allows splitting files across many directories rather than storing them all in one directory. While this [isn't a problem when using S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html), when using the local filesystem storing a large number of files in one directory could negatively affect performance.

| Format | Description | Example
|:--|:--|:---
|`"none"`| Doesn't create a directory structure, storing all uploads for a collection in a subdirectory of the `basePath` location |
|`"date"`| Creates a directory structure using parts derived from the current date | `2016/12/19/test.jpg`
|`"datetime"`| Creates a directory structure using parts derived from the current date and time | `2016/12/19/13/07/22/test.jpg`
|`"sha1/4"`| Splits SHA1 hash of the image's filename into 4 character chunks | `cb56/7524/77ca/e640/5f85/b131/872c/60d2/1b96/7c6a/test.jpg` |
|`"sha1/5"`| Splits SHA1 hash of the image's filename into 5 character chunks | `cb567/52477/cae64/05f85/b1318/72c60/d21b9/67c6a/test.jpg` |
|`"sha1/8"`| Splits SHA1 hash of the image's filename into 8 character chunks | `cb567524/77cae640/5f85b131/872c60d2/1b967c6a/test.jpg` |

## Authentication

Media upload requests must be authenticated with a Bearer token supplied in an `Authorization` header along with the POST request. See the [Authentication](authentication) section for details on obtaining a token.

## File Storage

API ships with two file storage handlers, one for storing files on the local filesystem and the other for storing files in an Amazon S3 bucket. If you need access to the files from another application, for example DADI CDN, we recommend using the S3 option.

## Configuring Amazon S3

If the S3 storage handler is used, an additional set of configuration properties are required as seen in the `s3` block below:

```json
"media": {
  "storage": "s3",
  "basePath": "uploads",
  "pathFormat": "date",
  "s3": {
    "accessKey": "asdf4hdhh422ss",
    "secretKey": "agDo40XIoK7jPxOx3HU9Pl",
    "bucketName": "media",
    "region": "eu-west-1"
  }
}
```

> **Security Note:** We don't recommend storing your AWS credentials in the configuration file. The `accessKey` and `secretKey` properties should instead be set as the environment variables `AWS_S3_ACCESS_KEY` and `AWS_S3_SECRET_KEY`.

## Filename clashes

If the filename of an file being uploaded is the same as an existing file, the new file will have it's name changed by adding the current timestamp:

* Existing filename: `test.jpg`
* New filename: `test-1480482847099.jpg`

## Uploading a file

To upload a file send a `multipart/form-data` POST to the media collection's endpoint. On successful upload the file's metadata is returned as JSON, and includes an identifier that can be used to create a reference to the file from another collection.

### Uploading a file with cURL

```bash
curl -X POST
  -H "Content-Type: multipart/form-data"
  -H "Authorization: Bearer 8df4a823-1e1e-4bc4-800c-97bb480ccbbe"
  -F "data=@test.jpg" "http://api.somedomain.tech/1.0/library/images"
```

### Uploading a file with Node.js

```js
var FormData = require('form-data')

var options = {
  host: 'api.somedomain.tech',
  port: 80,
  path: '/1.0/library/images',
  headers: {
    'Authorization': 'Bearer 8df4a823-1e1e-4bc4-800c-97bb480ccbbe',
    'Accept': 'application/json'
  }
}

var uploadResult

var form = new FormData()
form.append('file', fs.createReadStream(filePath))

form.submit(options, (err, response, body) => {
  if (err) return reject(err)

  response.on('data', (chunk) => {
    if (chunk) {
      uploadResult += chunk
    }
  })

  response.on('end', () => {
    // finished
    console.log(uploadResult)
  })
})
```

### Response

If successful, expect a response similar to the below examples.

#### Disk storage

```http
HTTP/1.1 201 Created
Server: DADI (API)
content-type: application/json
content-length: 305
Date: Mon, 19 Dec 2016 05:20:29 GMT
Connection: keep-alive

{
  "results":[
    {
      "fileName":"test.jpg",
      "mimetype":"image/jpeg",
      "width":1920,
      "height":1080,
      "path":"/Users/userName/api/workspace/media/2016/12/19/test.jpg",
      "contentLength":173685,
      "createdAt":1482124829485,
      "createdBy":"your-client-key",
      "v":1,
      "_id":"58576e1d5dd9975624b0d92c"
    }
  ]
}
```

#### Amazon S3 storage

```http
HTTP/1.1 201 Created
Server: DADI (API)
content-type: application/json
content-length: 305
Date: Mon, 19 Dec 2016 05:20:29 GMT
Connection: keep-alive

{
  "results":[
    {
      "fileName":"test.jpg",
      "mimetype":"image/jpeg",
      "width":1920,
      "height":1080,
      "path":"workspace/media/2016/12/19/test.jpg",
      "contentLength":173685,
      "awsUrl":"https://bucketName.s3.amazonaws.com/workspace/media/2016/12/19/test.jpg",
      "createdAt":1482124902978,
      "createdBy":"your-client-key",
      "v":1,
      "_id":"58576e72bafa53b625aebd4f"
    }
  ]
}
```

## Referencing files from another collection

Once a file is uploaded, it's identifier can be used to create a reference from another collection. For this example we have a collection called `books` with the following schema:

```json
{
  "fields": {
    "title": {
      "type": "String",
      "required": true
    },
    "content": {
      "type": "String",
      "required": true
    },
    "image": {
      "type": "Reference",
      "settings": {
        "collection": "images"
      }
    }
  },
  "settings": {
    "cache": true,
    "count": 40,
    "compose": true,
    "sort": "title",
    "sortOrder": 1
  }
}
```

The `image` field is a Reference field which will lookup the `images` collection to resolve the reference. Having uploaded an image file and received it's metadata, we can now send a POST to the `books` collection with the image identifier.

```http
POST /1.0/library/books HTTP/1.1
Host: api.somedomain.tech
content-type: application/json
Authorization: Bearer 8df4a823-1e1e-4bc4-800c-97bb480ccbbe

{
  "title": "Harry Potter and the Philosopher's Stone",
  "content": "Harry Potter and the Philosopher's Stone is the first novel in the Harry Potter series and J. K. Rowling's debut novel, first published in 1997 by Bloomsbury.",
  "image": "58576e72bafa53b625aebd4f"
}
```

A subsequent GET request for this book would return a response such as:

```json
{
  "title": "Harry Potter and the Philosopher's Stone",
  "content": "Harry Potter and the Philosopher's Stone is the first novel in the Harry Potter series and J. K. Rowling's debut novel, first published in 1997 by Bloomsbury.",
  "image": {
    "fileName":"test.jpg",
    "mimetype":"image/jpeg",
    "width":1920,
    "height":1080,
    "path":"workspace/media/2016/12/19/test.jpg",
    "contentLength":173685,
    "awsUrl":"https://bucketName.s3.amazonaws.com/workspace/media/2016/12/19/test.jpg",
    "createdAt":1482124902978,
    "createdBy":"your-client-key",
    "v":1,
    "_id":"58576e72bafa53b625aebd4f"
  }
}
```

## Signed URLs

A signed URL gives

### Configuration

```json
"media": {
  "enabled": true,
  "tokenSecret": "catboat-beatific-drizzle",
  "tokenExpiresIn": "10h"
}
```

### Request a signed URL
```
POST /api/media/sign

{
  fileName: 'test.jpg',
  mimetype: 'image/jpeg'
}
```

#### Response
```
{
  url: "/api/media/{token}"
}
```

### POST the file
```
POST /api/media/{token}

test.jpg
```



### Override the expiry when requesting a signed URL
```
POST /api/media/sign

{
  fileName: 'test.jpg',
  mimetype: 'image/jpeg',
  expiresIn: '15000' // value in seconds
}
```


Close #229

```json
"settings": {
  "type": "media",
  "signUploads": false
}
```

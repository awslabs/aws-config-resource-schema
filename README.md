## AWS Config Resource Schema

AWS Config resource property files define the properties and types of the AWS Config resource configuration items (CIs) that are searchable using the `SelectResources` API.  These files ease discovery of searchable properties and allow API users to more accurately craft queries suited for specific resource types.

Documentation on the `SelectResources` API can be found here: [Querying the Current Configuration State of AWS Resource](https://docs.aws.amazon.com/config/latest/developerguide/querying-AWS-resources.html)

This repository has the following directory structure:

```
└── config
    ├── properties
    │   ├── AWS.properties.json
    │   └── resource-types
    │       ├── AWS::ACM::Certificate.properties.json
    │       ├── AWS::AutoScaling::AutoScalingGroup.properties.json
    ...     ...
```

### Resource Properties

Resource property (`.properties.json`) files are JSON-encoded and have the following shape:

```json
{
  "...": "...",
  "sample.property.name": "string",
  "...": "...",
}
```

Here `sample.property.name` is the name of a property and `string` is its type.  Currently, the following types are supported:

* `boolean`: a Boolean value
* `cidr_block`: a CIDR block (e.g., `192.168.1.0/24`)
* `date`: a date/time instance
* `float`: a floating point value
* `integer`: an integer value
* `ip`: an IP address (e.g., `192.168.1.1`)
* `string`: a character sequence

Resource property files exist for each resource type Config supports; they are located in the [config/properties/resource-types](config/properties/resource-types) directory; they are named according to the corresponding Config resource type name (e.g., the properties file for resource type `AWS::EC2::Instance` is `AWS::EC2::Instance.properties.json`).  A merged property file containing resource properties for all AWS resource types is located in [config/properties/AWS.properties.json](config/properties/AWS.properties.json).

#### Example Usage 1

Assume we want to search for all S3 buckets having tag key `CostCenter`, and tag value `12345` in region `ap-northeast-1` with a name starting with `quicksilver`.  To find the corresponding properties, first open the [resource properties file for the resource type `AWS::S3::Bucket`](config/properties/resource-types/AWS::S3::Bucket.properties.json).  Therein, find the relevant properties:

```json
{
  "...": "...",
  "awsRegion": "string",
  "...": "...",
  "resourceName": "string",
  "resourceType": "string",
  "...": "...",
  "tags.tag": "string",
  "...": "...",
}
```
Then use them to craft the query:
```sql
SELECT resourceId WHERE resourceType='AWS::S3::Bucket' AND awsRegion='ap-northeast-1' AND resourceType LIKE 'quicksilver%' AND tags.tag='CostCenter=12345'
```

(Note that the `tags.tag` property is a concatenation of the `tags.key` and `tags.value` properties and makes it possible to search using both tag key and value components simultaneously.  For instance, if `tags.key` has the value `Stage` and `tags.value` has the value `Production`, `tags.tag` will have the value `Stage=Production` (concatenated with an `=` sign).)

#### Example Usage 2

Assume we want to count the number of EC2 instances having tag key `Stage` (and any tag value), in availability zone `us-east-1a` running AMI image ID `ami-12345`.  To find the corresponding properties, first open the [resource properties file for the resource type `AWS::EC2::Instance`](config/properties/resource-types/AWS::EC2::Instance.properties.json).  Therein, find the relevant properties:

```json
{
  "...": "...",
  "availabilityZone": "string",
  "...": "...",
  "configuration.imageId": "string",
  "...": "...",
  "resourceType": "string",
  "...": "...",
  "tags.key": "string",
  "...": "...",
}
```

Then use them to craft the query:
```sql
SELECT COUNT(resourceId) WHERE resourceType='AWS::EC2::Instance' AND availabilityZone='us-east-1a' AND configuration.imageId='ami-12345' AND tags.key='Stage'
```

(Note that, as mentioned above the `tags.key` property refers only to the key or "name" of the tag.)

## License

This library is licensed under the Apache 2.0 License.


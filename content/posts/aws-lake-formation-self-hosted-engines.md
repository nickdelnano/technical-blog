+++
date = '2024-09-15T00:00:00-08:00'
title = 'AWS Lake Formation with Self Hosted Query Engines via Apache Iceberg'
+++

[Lake Formation](localhost:1313) is a neat service that provides a better authorization abstraction for data lake objects than IAM. What do I mean by "better"?
- Friendlier for data roles (analysts, data engineers, data scientists) to understand and use
	- These roles work with data lake table names. They don't want to care about S3 paths.
- Enables automated permission audits. `aws lakeformation list-permissions` replaces ... parsing IAM policies?

With Lake Formation, IAM permissions on S3 (data) and Glue (metadata) resources become `GRANT` statements. If you have managed IAM change management in an enterprise, or even if you have written an IAM policy just one time, this will be a welcome change for you.

Bringing data lake permissions into the data world is a huge value add for the triad of data users, the data platform team and a security organization.

*But there are prerequisites, and hopefully your deployment meets them. If not, then maybe this post will help you.*

## Query Engine Compatibility
To enable [Lake Formation's implementation](https://docs.aws.amazon.com/lake-formation/latest/dg/how-it-works.html), it requires using [AWS hosted query engines](https://docs.aws.amazon.com/lake-formation/latest/dg/service-integrations.html). If you are self hosting your query engine, or if you are using an engine not supported in Lake Formation, you can implement the [credential vending API](https://docs.aws.amazon.com/lake-formation/latest/dg/using-cred-vending.html). It is possible – [vendors](https://docs.starburst.io/latest/security/aws-lake-formation.html) in the space have done this – but what if you don't want to, or you simply want an open source solution?

## Expanded Compatibility with Apache Iceberg
Enter Iceberg's [LakeFormationAwsClientFactory](https://github.com/apache/iceberg/blob/main/aws/src/main/java/org/apache/iceberg/aws/lakeformation/LakeFormationAwsClientFactory.java#L51) class.

This was [contributed by AWS](https://github.com/apache/iceberg/pull/4280) -- plausibly for use with their query engine services -- but it works in the case of a single party. See example configuration [here](https://github.com/apache/iceberg/issues/10226).

Now you can integrate any Iceberg compatible query engine with Lake Formation permissions. For my case this was open source Spark and Flink.

There is an important limitation:  Lake Formation row and column security policies cannot be securely applied. As part of the [credential vending contract](https://docs.aws.amazon.com/lake-formation/latest/dg/roles-and-responsibilities.html), the third-party must not leak credentials and must return only filtered data to users. This is not applicable in this case since there is only one party and the application will access the credentials. If this limitation is not a blocker for you, then you can enjoy Lake Formation support in all of your query engines.

## Addendum
The Lake Formation implementation in Iceberg does not work to create tables. The integration test in the repo fails (and is not run on CI? Probably because it requires an AWS account with broad permissions on the IAM service).

I've filed [issue #10226](https://github.com/apache/iceberg/issues/10226) about this.

This can be worked around
- Create a table using Glue APIs. My use-case is for client IAMs to have no S3 permissions and only Lake Formation grants, so I use `glue create-table` with [special parameters](https://docs.aws.amazon.com/lake-formation/latest/dg/creating-iceberg-tables.html).
- Run DDL through a separate Spark/Flink catalog instance without the Lake Formation properties.

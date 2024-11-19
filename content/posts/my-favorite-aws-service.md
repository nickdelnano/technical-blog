+++
date = '2024-08-20T00:00:00-08:00'
title = 'My Favorite AWS Service'
+++

[CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)

But not with the console UI. Send it to [CloudTrail Lake](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-lake.html). Or send it your log search tool like Splunk.

CloudTrail has saved me so much time and made it possible to answer questions that are otherwise very difficult, like
- Debugging behavior of roles that an AWS service uses. Recently this was setting up a service linked role for Lake Formation, and a data platform where IAMs exclusively use Lake Formation permissions instead of S3 permissions (data users, security and auditing teams love this!)
- Check usage of a specific resource to understand its usage or to deprecate it (IAM role, Glue table, DynamoDB table, etc)
- Quickly understanding how an application uses AWS resources without reading any code.

I use it to debug issues almost daily, and I've seen it up-level many engineer's debugging skills.

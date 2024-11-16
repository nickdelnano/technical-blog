+++
date = '2024-01-08T00:00:00-08:00'
title = 'Redshift DBA Handbook'
+++

I became a Redshift DBA by chance but have grown to like the product and the role. At first I found Redshift to be opaque and challenging to debug. The standard system views leave some questions unanswered. Here's my handbook of queries that fills in some of the gaps.

I haven't found much writing about these types of queries so I thought I would contribute to it.

This is from my experience with only provisioned clusters. Some aspects of Redshift Serverless are surely different.

## Cluster State
[Current queries (running and queued)](https://github.com/awslabs/amazon-redshift-utils/blob/master/src/AdminScripts/running_queues.sql)
If you use Redshift, this query should be near first item to check in a runbook for "Redshift cluster is slow".
It provides a per user, query, queue breakdown of what is happening in your cluster. Run this in a sql client that supports sorting results by column without re-running the query, and you can go column by column to determine what use case is halting your cluster.

Common Redshift failure modes that this view easily identifies:
- One query using a ton of CPU time and blocking the whole workload (Redshift is very vulnerable to this)
- One use case running many, many queries and backlogging a WLM queue (maybe a Spark application running with too many executors)
- A query hopelessly spilling memory to disk, that is unlikely to exceed

## Blocking locks
[Get blocking locks](https://github.com/awslabs/amazon-redshift-utils/blob/master/src/AdminViews/v_get_blocking_locks.sql)

This view is helpful to see any processes that are blocked on others. See [here](https://repost.aws/knowledge-center/prevent-locks-blocking-queries-redshift) for Redshift's lock types.

## Resource usage
[SVL_QUERY_METRICS_SUMMARY](https://docs.aws.amazon.com/redshift/latest/dg/r_SVL_QUERY_METRICS_SUMMARY.html)

This perspective shows resource consumption per use case. `query_cpu_time` is a good indicator, however, this metric is not tracked in any system table for concurrency scaling clusters. If your workload relies on concurrency scaling then you are limited in how you can attribute resource usage. Query execution time is the most capable metric that is available in concurrency scaling but it is flawed – it's easily affected by concurrent cluster workload and concurrency scaling is an opaque box. Resource attribution is an unsolved problem in provisioned clusters but it has improved in the Serverless product.

## Permissions
[SVV_RELATION_PRIVILEGES](https://docs.aws.amazon.com/redshift/latest/dg/r_SVV_RELATION_PRIVILEGES.html)

[SVV_SCHEMA_PRIVILEGES](https://docs.aws.amazon.com/redshift/latest/dg/r_SVV_SCHEMA_PRIVILEGES.html)

These views were released sometime in 2023 and are a major improvement over the previous solutions.
Take them and join with a query that [joins users and groups](https://stackoverflow.com/questions/51913807/redshift-how-to-list-all-users-in-a-group) to see what users have (read, write) access to what tables and whether it's through a group or per user grant.

# Hope this helps :)
I wish I had this all when I started working with Redshift, so I hope it helps you.

A future topic on this blog will hopefully be a better default Redshift cloudwatch dashboard than the one Cloudwatch provides, but for now konw that theres a ton more Cloudwatch metrics, especially ones for WLM queues!

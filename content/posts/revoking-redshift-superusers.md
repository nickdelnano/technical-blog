+++
date = '2024-04-02T00:00:00-08:00'
title = 'Revoking Redshift Superusers'
+++

In recent years I have been tasked with administering a fleet of Redshift clusters. Once upon a time I was tasked with reducing the number of admin level users, or [superusers](https://docs.aws.amazon.com/redshift/latest/dg/r_superusers.html) as Redshift calls them. If you find yourself in a similar situation, know that there is a trap waiting for you. Now you will be aware of it.

If your organization has ever granted superuser access in any non-least privilege manner, you will likely have views owned by these users. Hopefully someday you apply least privilege to this part of your data infrastructure as Redshift superuser has access to everything within the cluster and likely parts of your data lake.

**Before revoking superuser access for user X, confirm that X has SELECT privileges on all dependent objects in the views it owns.** Otherwise it is relying on the superuser privilege for this, and without it, queries from any user on the view will fail.

> [42501] ERROR: permission denied for relation view_name

That is - permission denied for the view owner! Not the user of *select current_user;*

Non-intuitive, right? The [CREATE VIEW page](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_VIEW.html) of the manual is missing some content; it mentions the requirement for late-binding views but it applies to regular views too.

I prefer a convention that all views are owned by a service user or the cluster root user. Data users submit DDL through a code reviewed process and the data platform team (or their automation) executes the DDL, including setting an appropriate owner.

If you trace through the system tables or the user activity log, you will see what Redshift is using the view owner user for.


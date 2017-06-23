# aws-setup-s3-accesslogs

This project provisions AWS infrastructure to hold access logs for
various services, primarily S3 bucket logs.

S3 bucket logging has some weird quirks, like how a bucket can only
provide access logs to another bucket in the same region.  Thus, this
project ensures that the same infrastructure exists in each region, and
that any dependent resources are region-specific so one AWS region
outage does not prohibit access logs from being created or monitored.

## Design Goals

1. Bucket logging must work in each region (it is region-locked).
1. Access logs must be stored in an organized manner.
1. Bucket storing access logs must itself log access to itself.
1. Long-term storage and cost must be kept to a minimum.
1. Infrastructure must be tagged for appropriate cost control.

## Requirements

* Ansible 2.2+
* AWS account
* Permissions to create/update/delete: CloudFormation stacks, S3 buckets,
SNS topics.

## Workflow

### Launching the Stack

Ansible is used to orchestrate the creation and deletion of the AWS
infrastructure (via CloudFormation), and to ensure that the stacks
exist in all available AWS regions.

To create or update the stack, perform the following:

```
ansible-playbook -i localhost.inventory -e "name_prefix=SOMETHING" create-stack.yml
```

The `name_prefix` parameter is special and must be specified by the caller.
It will form part of the S3 bucket name, so it must be something globally
unique (not just unique to yourself). 

**Do not commit the name prefix to a public repository, lest someone is
informed about the bucket where you store your logs.**

### Deleting the Stack

There is a corresponding [`delete-stack.yml`](delete-stack.yml) file that
will remove the project's CloudFormation stack (and all resources) from
every region.

## Security Controls

### Accidental Stack Update

There exists a stack policy that denies all updates.  This stack is
expected to exist for a long time, and there should not be many
updates to it.  As a result, any updates to this stack will fail
because of the stack policy violation.

If an update is warranted, update the stack's policy to allow updates
to all (or specific) resources.  Once the update operation completes
successfully, the original stack policy (denying all updates) will be
placed on the stack again.

NB: Stack policies do not stop CloudFormation stacks from being deleted,
it only protects underlying resources.

NB: Stack policies do not stop underlying resources from being changed
manually (through their respective service endpoint), it only protects
against CloudFormation updating the resource.

### Accidental Stack Deletion

If you attempt to delete the stack and there exists data in it, the
operation will fail and CloudFormation will report that the bucket is
not empty.  This is a safety mechanism.  To proceed, empty the bucket
manually and then re-run the orchestration.

### Abnormal Event Notification

As this bucket is storing audit logs for the long-term, an SNS topic
exists to provide notifications when an object is removed from the
bucket.  However, there is no corresponding subscriber on the SNS
topic, as this is up to the caller to provide in another stack.  A
stack export is available providing the ARN of the SNS topic.

## License

tl;dr MIT License

See [LICENSE](LICENSE) for more details.


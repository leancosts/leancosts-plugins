---
name: connect
description: Connect a cloud to leancosts from the terminal and watch the first sync land: check where setup stands, run the read-only grant with the user's own cloud CLI, create the connection, and confirm the first sync. Use when the user says "connect my AWS account to leancosts", "connect Azure to leancosts", "set up leancosts", or asks what is still missing in leancosts setup.
---

# Connect a cloud to leancosts

leancosts only ever reads your cloud. The grant is read-only and the user runs it with their own CLI. No secret,
access key or service-account key passes through this conversation. Never ask for one. **One cloud per invocation.**
If any step fails, show the error, call `get_sync_logs` when a connection exists, and stop.

## 1. Where setup stands

Call `get_onboarding_status`. Show what is already connected, its state and any last error. Then ask which cloud to
connect, unless the user has already said.

## 2. Check whose cloud this is

Show the identity the user's CLI will act as and ask them to confirm it is the right account:

- Azure: `az account show --query "{name:name, id:id, tenant:tenantId}"`
- AWS: `aws sts get-caller-identity`
- GCP: `gcloud config list account project`

## 3. Grant and connect

**Azure.** Call `start_azure_oauth` with the tenant id from step 2. Show the `verificationUri` and `userCode`, and
tell the user to sign in there. Then poll `get_azure_oauth_status` until it finishes. leancosts creates the
read-only app registration, the roles and the connection. The user signing in needs Owner or User Access
Administrator on the subscription. If they do not have it, offer `request_admin_handoff` instead, then stop.

**AWS.** Call `get_grant_artifacts` (`provider: "aws"`). If `payerRoleStack.launchUrl` is set, the simplest path is
to give the user that link to open signed in to the payer account (it creates the same stack); then continue from
the `RoleArn` output as below. Otherwise write `payerRoleStack.template` to
`payerRoleStack.fileName`, show the user the role it creates, and ask "Create this stack in <account>?". Only after
a literal yes, run `payerRoleStack.deployCommand` there. Then call `create_aws_connection` with the `RoleArn` the
command printed and the `externalId` from the artifacts. For an Organization, `memberRoleStack` is the StackSet
template for the member accounts; the user deploys it from the management account and sets it under Member role in
the web app.

**GCP.** Call `get_grant_artifacts` (`provider: "gcp"`), show the commands, and run them only after a literal yes.
There is no tool to create a GCP connection. Send the user to Admin → Connections → Add connection → Google Cloud in the web app to finish it,
then continue from step 4.

**OCI.** Call `create_oci_connection` with the tenancy OCID and the home region. The OCID of the read-only user is
optional (a trial still needs it): when the user is created by a Terraform file, its OCID is the user_ocid output
after the apply, so create the connection, let the user apply, call `set_oci_user_ocid` with that OCID (it is set once),
then `test_connection`. A test refuses until the user OCID is set. The create answers the public key and the policy
statements. To show them again later, call `get_grant_artifacts`
(`provider: "oci"`, plus `connectionId` when the organization holds more than one connection). The user pastes the
policy statements and uploads the public key in the OCI console; no command runs here. When the customer's
repository already uses Terraform, use the `terraform` field of the create and `get_grant_artifacts` answers instead
(a connected file and a child-tenancy file, each alone in its own directory; a trial gets none): the user applies the
connected file, you call `set_oci_user_ocid` with its user_ocid output, and each child tenancy's owner applies the
child file with their own profile. Then continue from step 4.

## 4. First sync

Call `test_connection`, then `trigger_sync`. Poll `get_sync_progress` until the sync finishes. If it fails, call
`get_sync_logs` and show the lines that explain it.

End with one line: which account is now connected and when its first costs will show.

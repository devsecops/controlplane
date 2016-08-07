# Separation of Duties (SoD) and Toxic Combinations

[Separation of Duties] (http://www.sans.edu/research/security-laboratory/article/it-separation-duties) is an internal controls concept, primarily from financial auditing.  But it is similarly applicable in technology.

The classic example of a case where Separation of Duties is required is in the Accounts Payable (AP) process.  The Accounts Payable process consists of the following major steps:
1. Receive a bill to be paid.
2. Authorize the bill to be paid.
3. Pay the bill.

Steps 2 & 3 are toxic if they are both controlled by the same person/entity because it would be a simple matter to forge an invoice, authorize payment of the invoice, and pay the invoice.  This would be Accounts Payable fraud.

Similarly, in IT systems, the Identity Provisioning and Access Granting processes present the potential for toxic combination.  Consider the following:
1. Ability to create an identity (userID).
2. Ability to grant privileges to an identity.

If you possess both of these abilities, then you likely have the ability to grant yourself or another actor full administrative privileges within the IT system.  Not only is this a toxic combination, it could make you a target for credential theft or even coercion.  This is why Role Engineering to avoid toxic combinations and apply the principle of Separation of Duties is so important.


   * Max Ramsay from AWS discusses considerations for how to apply Separation of Duties (SoD) (here)(https://blogs.aws.amazon.com/security/post/TxQYSWLSAPYVGT/Guidelines-for-When-to-Use-Accounts-Users-and-Groups)
   * Separation of Duties is an internal controls concept, primarily from financial auditing.  But it is equally applicable in technology.
   * [Another good reference on what SoD is](http://szabo.best.vwh.net/separationofduties.html)


## Toxic Actions

**Note:** This collection is likely incomplete, and could be easier to understand.  Please contribute to help make it better.

### Toxic individual Actions

The idea is that these individual actions are so potentially powerful that they should never be granted, or must be constrained, controlled, and monitored very carefully.

* `iam:Put*` - _These actions allow the creation and update/overwrite of an Access Policy that is 'inline' to a Group, Role, or User.  Until the advent of 'Managed Policies', they were the only way to grant privilege to a Group, Role, or User.  Unfortunately, they can be used to grant elevated privilege and they take effect immediately.  So, any entity that has the privilege to use these actions is likely to be able to elevate itself to a full administrator_ (`"Action":"*", "Resource": "*"`) _unless very carefully managed.  This is why this family of actions is inherently toxic and does not require being combined with other actions to become toxic._
  * `iam:PutGroupPolicy`
  * `iam:PutRolePolicy`
  * `iam:PutUserPolicy`
* `iam:Attach*` - _These actions allow an existing Managed Policy to be attached to an existing Group, Role, or User.  AWS provide a catalog of standard Managed Policies that they manage for you, but this catalog includes policies that grant full administrative control over the account_ (`"Action":"*", "Resource": "*"`)_, and others that grand full IAM administrative control_ (`"Action":"iam:*", "Resource": "*"`) _unless very carefully managed.  This is why this family of actions is inherently toxic and does not require being combined with other actions to become toxic._



### Toxic Combinations

The idea is that toxic combinations should never be granted to an entity (human, automated process, etc.), they should be split apart and two separate entities should have to coordinate action to achieve the task that requires the toxic combination.

### To Do - Work through these to detail how/why they are toxic:
* `iam:Change*` -
* `iam:Create*` -
* `iam:Deactivate*` -
* `iam:Delete*` -
* `iam:Detach*` -
* `iam:Enable*` -
* `iam:Remove*` -
* `iam:Resync*` -
* `iam:Set*` -
* `iam:Update*` -
* `iam:Upload*` -

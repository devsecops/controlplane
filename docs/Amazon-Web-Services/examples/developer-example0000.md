# Example developer / engineer / DevOps type role(s)

Developers, engineers, and other productivity and problem solving oriented people want to do their work without too much (any!) friction.  This is totally understandable.  So, how do we construct an AWS IAM Access policy that mostly gets out of the way, while preventing the worst of the toxic combinations of AWS API Actions that allow for privilege escalation?  These examples hope to provide a sane starting point from which to develop your own AWS IAM Policies, and we hope that you'll contribute back examples of your own.

**Note 1:** An explanation of each policy block will be provided below the example.

**Note 2:** These are just access policy examples and they have to be attached, either inline or via managed policies, to a Group, Role, or User to be used.

## developer-example0000
_(example 0, 'cause we had to start somewhere...)_

This IAM Access Policy uses five (5) policy blocks to accomplish the following:

1. Provide a list of **whitelisted** AWS services that _may_ be used, and effectively blacklist any service that is not whitelisted.
2. Explicitly grant a few fundamental IAM actions that you just have to have to function, but omit any of the toxic actions.
3. Explicitly deny toxic API actions.
4. Explicitly grant the `iam:PassRole` action on a short list of IAM Roles for EC2.
5. Grant all actions that are not IAM actions that have not otherwise been denied.

This will require a separate role and workflow to to the IAM Role engineering to provide the IAM Roles that may be needed by a particular project, but will

###

This policy should be attached to a Role as an inline or managed policy.  A managed policy is probably better for a variety of reasons that are likely explored elsewhere.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "whitelistServicesThatAreAllowedToBeUsed",
      "NotAction": [
        "iam:*",
        "s3:*",
        "ec2*",
        "elasticloadbalancing:*",
        "cloudwatch:*",
        "rds:*",
        "sns:*"
      ],
      "Effect": "Deny",
      "Resource": [
        "*"
      ]
    },
    {
      "Sid": "aFewIamActionsThatYouProbablyReallyNeedToFunction",
      "Action": [
        "iam:GetPolicy",
        "iam:GetPolicyVersion",
        "iam:GetRole",
        "iam:GetRolePolicy",
        "iam:ListAccountAliases",
        "iam:ListAttachedRolePolicies",
        "iam:ListEntitiesForPolicy",
        "iam:ListInstanceProfiles",
        "iam:ListInstanceProfilesForRole",
        "iam:ListPolicies",
        "iam:ListPoliciesGrantingServiceAccess",
        "iam:ListPolicyVersions",
        "iam:ListRolePolicies",
        "iam:ListRoles"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "denyToxicActions",
      "Action": [
        "iam:Put*",
        "iam:Attach*",
        "iam:Change*",
        "iam:Create*",
        "iam:Deactivate*",
        "iam:Delete*",
        "iam:Detach*",
        "iam:Enable*",
        "iam:Remove*",
        "iam:Resync*",
        "iam:Set*",
        "iam:Update*",
        "iam:Upload*"
      ],
      "Effect": "Deny",
      "Resource": [
        "*"
      ]
    },
    {
      "Sid": "managePassRoleVeryTightlyToPreventPrivilegeEscalation",
      "Action": [
        "iam:PassRole"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:::Resource1",
        "arn:::Resource2"
      ]
    },
    {
      "Sid": "actuallyBeAbleToDoStuff",
      "NotAction": [
        "iam:*"
      ],
      "Effect": "Allow",
      "Resource": [
        "*"
      ]
    }
  ]
}
```

### 1. whitelistServicesThatAreAllowedToBeUsed

Literally, provide a list of AWS services that are allowed to be used.  I generally don't recommend the use of the `NotAction` keyword because parsing the logic is harder for humans, but, in this case, it is an elegant solution to a large and changing problem-space: namely that AWS are constantly adding services and updating the actions that can be managed via IAM.  New AWS services are often MVPs (minimum viable products), and their definition of 'viable' may not meet the minimum standards for your business.  The result is that you need a way to throttle the adoption/use of new services without having to constantly chase the list of services to prohibit.

This policy block explicitly denies any action not covered by this list, and leaves all other actions implicitly denied (i.e. they still have to be granted some other way).  Since an explicit deny overrides an explicit allow, this is a simple and powerful way to enforce an whitelist.  You'll rapidly find that AWS is pretty near useless without a core set of services, so you won't be able to limit this quite as much as you might initially think (i.e. if you want AutoScale to work, you need CloudWatch, and if you ever want CloudWatch to notify you of anything, you need SNS, and it is really hard to exist without S3 since it was the first service and many of the others pre-suppose its availability).

#### Why didn't you include ???

* `sts:*` - don't you need that to assume roles as part of a separation of duties and least privilege model?
  * Yes, you do. But not in the access policies that are associated with a Role that you assume.  That would potentially allow you to assume another role and 'hop-roles'.
* `kms:*` - don't you want your developers and engineers to to use KMS for encryption?
  * Yes, of course.  But these actions are primarily about key _management_ and the policies that are attached to the keys define who/what can use them to decrypt data.  You probably want service roles to have privilege to use specific KMS keys, but there is little need for a human to be able to use the keys.

### 2. aFewIamActionsThatYouProbablyReallyNeedToFunction

It turns out that you can't simply prohibit all IAM actions, even though that seems like a logical first response to preventing account takeover.  For example, a developer, engineer or dev/ops person probably needs the ability to run an instance, and tie that instance to an Instance Profile so that the instance has an IAM Role to be able to interact with other AWS services (maybe it needs to retrieve a deployment artifact from S3 during bootstrap).  This also means that the human probably needs to be able to peruse the list IAM Roles that are available to select the correct Role.

### 3. denyToxicActions

These are the major actions in IAM that can lead to an attacker elevating privilege.  More info in [Separation of Duties and Toxic Combinations](../references/separation-of-duties-and-toxic-combinations.md).

### 4. managePassRoleVeryTightlyToPreventPrivilegeEscalation

The `iam:PassRole` action is what allows the entity launching an EC2 Instance to associate a specific IAM Role with the instance.  If a CloudFormation template is used to launch a stack that requires a particular IAM Role, and the entity launching the CloudFormaiton does not have the correct `PassRole` privilege, the CloudFormation stack will fail.  However, the ability to associate an IAM Role with an EC2 instance creates an opportunity to elevate privilege by launching an instance with a Role that has administrative permissions (`"Action": "*", "Resource": "*"`). Thus, the Roles that can be

### 5. actuallyBeAbleToDoStuff

This section uses the `NotAction` keyword again to actually allow you to do stuff because it is paired with the `"Effect": "Allow"` statement.  Again, in this case `NotAction` provides an elegant solution to a large and changing problem-space: namely that AWS are constantly adding services and updating the actions that can be managed via IAM.

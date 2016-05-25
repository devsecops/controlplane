#Control Plane + Target Account(s)#

The Control Plane pattern is built on the [AWS IAM Cross-Account Roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/walkthru_cross-account-with-roles.html) technology and enables native access management for Humans and Applications interacting with AWS.

---

Guiding Principles for this pattern:

* Native use of AWS
* Blast Radius Containment
* Privileged Access Management
* Least Privilege




##Basic Structure

The basic Control Plane pattern has a single or primary control plane and one or more target accounts that have a trust relationship with the control plane. Long-Term Credentials associated with IAM Users are routinely in use in the primary control plane. Human access is brokered with MFA, and app access via Long-Term Credentials implements compensating controls from the various conditions statements available in IAM Policy.

An enhanced Control Plane pattern includes a second backup or recovery control plane, and each of the target accounts also has a trust relationship with the backup control plane. Minimal Long-Term credentials exist (only enough to seed-access to the backup control plane), and these credentials are stored securely for 'break-glass' scenarios.

###Image 1: Control Plane to Target(s) Relationship

Trust is delegated in from an IAM Role in the _trusting_ account to an IAM Principle in the _trusted_ account.  This trust is granular, meaning that specific IAM Roles in the _trusting_ account trust a specific IAM Principle in the _trusted_ account.  This is not an account-to-account trust (no so such model exists in AWS).

![1_control-target-relationship](/docs/img/1_control-target-relationship.png)


##AssumeRole Flows

When an IAM User wishes to access the permissions associated with a specific IAM Role, they use the AssumeRole API action on the STS sservice. These flow diagrams illustrate the flow of API actions initiated by the IAM user for each scenario.

###Image 2: Human AssumeRole Flow - Primary Control Plane

1. Present valid Long-Term credentials + MFA token code to request an AssumeRole action for a Central Role that can assume the Target Role.
2. Receive Temporary Security Credential that has permission to assume the Target Role.
3. Present Temporary Security Credential from step 2 to request an AssumeRole action for the Target Role.
4. Receive Temporary Security Credential from Target Role.  This credential will have the permissions of the Target Role.
5. Use the Temporary Security Credential from step 4 to perform actions authorized by the Target Role in the target account

![2_iam-sts-human-flow](/docs/img/2_iam-sts-human-flow.png)


###Image 3: Human AssumeRole Flow - Backup Control Plane

1. Retrieve valid Long-Term credentials + MFA token code generator from secure storage
2. Follow the previous flow.

![3_iam-sts-human-backup-flow](/docs/img/3_iam-sts-human-backup-flow.png)

##Putting it together

This diagram shows the primary and backup IAM Flows along with the Control Plane and Target Accounts.

![4_iam-sts-flows-together](/docs/img/4_iam-sts-flows-together.png)

##Role Policy Breakdown

The following sections provide illustrations and sample JSON policies for each of the AWS IAM Roles discussed in this pattern.

###Human Role: Control Plane (Central Role)

This is first IAM Role assumed via the AssumeRole action in the Human AssumeRole Flow above.

* This role trusts the 'root' principle of the control plane account.  This means that acess to assume this role is managed by the Access Policies associated with the other IAM Principals (IAM Users & IAM Roles) in the control plane account.
* This role enforces a condition requiring that a valid MFA token code be presented to assume the role.  Since MFA seeds are associated with IAM Users, this effectively limits the AssumeRole action to IAM Users.
* IAM Groups are used to manage the IAM Policies associated with IAM Users, and specific IAM Groups grant access to assume specific IAM Roles.
* HTTPS must be used on the API call to assume this role.
* This role grants the ability to assume a specific target role on any target account.

![5_central-human-role](/docs/img/5_central-human-role.png)

####Role Trust Policy

```
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<TrustedAccount>:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "Null": {
          "aws:MultiFactorAuthAge": false
        }
      }
    }
  ]
}

```

####Role Access Policy

```
{
  "Statement": [
    {
      "Action": [
        "sts:AssumeRole"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "true"
        }
      },
      "Effect": "Allow",
      "Resource": "arn:aws:iam::*:role/TargetPath/TargetRole”
    }
  ]
}

```

###Human Role: Target Account(s) (Target Role)

This is second IAM Role assumed via the AssumeRole action in the Human AssumeRole Flow above.

* This role trusts a specific IAM Role in the control plane account.  No other entity can assume this role.
* Since the only path to assume this role is from the specific role in the control plane account and that role enforces use of MFA, this role is only available with MFA (transitive property).
* This role should enforce the principles of Least Privilege & Separation of Duties, and only grant the minimum AWS API Actions required.

![6_target-human-role](/docs/img/6_target-human-role.png)

####Role Trust Policy

```
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<TrustedAccount>:role/CentralPath/CentralRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```

####Role Access Policy

```
{
  "Statement": [
    {
      "Action": [
        <Array of API Actions/Permissions>
      ],
      "Condition": {
        <Optional Conditions>
      },
      "Effect": "Allow",
      "Resource": [
        <Array of Resources>
      ]
    }
  ]
}

```


###App Role: Control Plane (Central Role)

This is the App version of first IAM Role assumed via the AssumeRole action in the Human AssumeRole Flow above (to do: add an App Flow).

* This role trusts a specific IAM User principle of the control plane account.  This means that acess to assume this role is managed by the Role Trust Policy of this role.
* This role enforces a set of conditions.
* HTTPS must be used on the API call to assume this role.
* This role grants the ability to assume a specific target role on any target account.

![7_central_iam-app-role](/docs/img/7_central_iam-app-role.png)

####Role Trust Policy

```
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<TrustedAccount>:user/UserPath/TrustedUser"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {"sts:ExternalId": <UUID String>},
        "Bool": {"aws:SecureTransport": true},
        "IpAddress": {"aws:SourceIp": [
            <Array of CIDR Blocks>
          ]}
    }}]
}

```

####Role Access Policy

```
{
  "Statement": [
    {
      "Action": [
        "sts:AssumeRole"
      ],
      "Condition": {
        "StringEquals": {"sts:ExternalId": <UUID String>},
        "Bool": {"aws:SecureTransport": true},
        "IpAddress": {"aws:SourceIp": [
            <Array of CIDR Blocks>
          ]}
      },
      "Effect": "Allow",
      "Resource": "arn:aws:iam::<TargetAccount>:role/TargetPath/TargetRole”
    }
  ]
}

```


###App Role: Target Account(s) (Target Role)

This is the App version of the second IAM Role assumed via the AssumeRole action in the Human AssumeRole Flow above.

* This role trusts a specific IAM Role in the control plane account.  No other entity can assume this role.
* The Role Trust Policy and the Role Access Policy documents enforce a set of conditions that constrain its use.
* This role should enforce the principles of Least Privilege & Separation of Duties, and only grant the minimum AWS API Actions required.

![8_target-iam-app-role](/docs/img/8_target-iam-app-role.png)

####Role Trust Policy

```
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<TrustedAccount>:role/CentralPath/CentralRole"
      },
      "Action": "sts:AssumeRole”,
      "Condition": {
        "StringEquals": {"sts:ExternalId": <UUID String>},
        "Bool": {"aws:SecureTransport": true},
        "IpAddress": {"aws:SourceIp": [
            <Array of CIDR Blocks>
          ]}
  }}]
}

```

####Role Access Policy

```
{
  "Statement": [
    {
      "Action": [
        <Array of API Actions/Permissions>
      ],
      "Effect": "Allow",
      "Resource": [
        <Array of Resources>
      ],
      "Condition": {
        "StringEquals": {"sts:ExternalId": <UUID String>},
        "Bool": {"aws:SecureTransport": true},
        "IpAddress": {"aws:SourceIp": [
            <Array of CIDR Blocks>
          ]}
  }}]
}

```

#Appendix

##Examples##

Some examples of this pattern include:

* [Placeholder]

##References##

* [References](docs/references)

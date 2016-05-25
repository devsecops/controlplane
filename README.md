#Control Plane + Target Account(s)

The Control Plane pattern allows for relative ease of use while balancing security needs such as, blast radius containment, minimal attack surface, privileged access management, and least privilege.

---

Guiding Principles for this pattern:

* Native use of Cloud Provider
* Blast Radius Containment
* Minimize Attack Surface
* Privileged Access Management
* Least Privilege




##Basic Structure

The basic Control Plane pattern has a single or primary control plane and one or more target accounts that have a trust relationship with the control plane. Long-Term Credentials associated with Users are routinely in use in the primary control plane. Human access is brokered with MFA, and app access via Long-Term Credentials implements compensating controls.

An enhanced Control Plane pattern includes a second backup or recovery control plane, and each of the target accounts also has a trust relationship with the backup control plane. Minimal Long-Term credentials exist (only enough to seed-access to the backup control plane), and these credentials are stored securely for 'break-glass' scenarios.

###Image 1: Control Plane to Target(s) Relationship

Trust is delegated from a Principal Entity in a _trusting_ account to a Principal Entity in the _trusted_ account.  This trust is granular, meaning that a specific Principal Entity in the _trusting_ account trusts a specific Principal Entity in the _trusted_ account.  This is not an account-to-account trust (such a broad trust is likely to introduce a design flaw that would allow elevation of privilege).

![1_control-target-relationship](/docs/img/1_control-target-relationship.png)

***Note:*** Diagram to be updated to be generalized to any Cloud Provider ([Issue #3](https://github.com/devsecops/controlplane/issues/3)).

##Examples

Some examples of this pattern include:

* [Amazon Web Services](/docs/Amazon-Web-Services/)
* [Google Cloud] (to be added: [Issue #4](https://github.com/devsecops/controlplane/issues/4))
* [Microsoft Azure] (to be added: [Issue #5](https://github.com/devsecops/controlplane/issues/5))

#Appendix

##References

* [References](/docs/references)

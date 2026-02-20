# Jenkins Enumeration

{% hint style="info" %}
## Jenkins Instances

Jenkins is often installed on Windows servers running as the all-powerful SYSTEM account.

If we can gain access via Jenkins and gain remote code execution as the SYSTEM account, we would have a foothold in Active Directory to begin enumeration of the domain environment.
{% endhint %}

## Discovery & Footprinting

Jenkins runs on Tomcat port `8080` by default. It also utilises port `5000` to attach slave servers.This port is used to communicate between masters and slaves.

Jenkins can use a local database, LDAP, Unix user database, delegate security to a servlet container, or use no authentication at all. Administrators can also allow or disallow users from creating accounts.

### Default Login Page

The default installation typically uses Jenkins’ database to store credentials and does not allow users to register an account. We can fingerprint Jenkins quickly by the telltale login page.

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

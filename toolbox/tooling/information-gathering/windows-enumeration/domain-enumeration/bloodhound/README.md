# BloodHound

{% hint style="warning" %}
#### Community Edition

This article and its sub-pages are referencing the Community Edition of BloodHound, NOT the Legacy version.
{% endhint %}

> BloodHound uses graph theory to reveal hidden and often unintended relationships within Active Directory, Entra ID (formerly Azure AD), and Microsoft Azure IaaS. Defenders (blue teams) and attackers (red teams) use BloodHound for a deeper understanding of privileged relationships in an environment.

{% hint style="info" %}
#### Download & Install

Install via apt:

`sudo apt install bloodhound -y`
{% endhint %}

***

## Setup

### Via Packages

{% stepper %}
{% step %}
### Install Bloodhound

Run the below command to install Bloodhound:

```bash
sudo apt install bloodhound -y
```
{% endstep %}

{% step %}
### Run the Config Script

After the installation is complete, run the installation script:

```bash
sudo bloodhound-setup
```

This will initialise the necessary services and configurations.
{% endstep %}

{% step %}
### Connect to the Web UI

Open a browser and navigate to [http://localhost:7474](http://localhost:7474/), then login using the default credentials:

```
username: neo4j
password: neo4j
```

Then complete the configuration.
{% endstep %}

{% step %}
### Update the API Config File

Now that we've updated the Neo4j password, update the BloodHound API config file to reflect the change (replace the password `"secret": "neo4j"`).

```bash
sudo nano /etc/bhapi/bhapi.json
```
{% endstep %}

{% step %}
### Start Bloodhound

Bloodhound can now be start using the below command:

```bash
sudo bloodhound
```

Default credentials:

```
username: admin
password: admin
```

When start for the first time, Bloodhound will ask for a new password to be set.
{% endstep %}
{% endstepper %}

{% hint style="info" %}
#### Password Reset

To reset Bloodhounds admin credentials run:

`sudo env bhe_recreate_default_admin=true bloodhound`
{% endhint %}

***

## Fixes

### PostgreSQL 'collation version mismatch'

{% embed url="https://www.kali.org/docs/troubleshooting/postgresql-collation-mismatch-error/" %}

{% code overflow="wrap" %}
```bash
sudo runuser -u postgres -- psql -c 'ALTER DATABASE postgres REFRESH COLLATION VERSION; ALTER DATABASE template1 REFRESH COLLATION VERSION;'
```
{% endcode %}

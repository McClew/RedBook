# Drupal Enumeration

## Discovery & Footprinting

### Public Files

A Drupal website can be identified in several ways, including by the header or footer message "**Powered by Drupal"**, the standard Drupal logo, the presence of a `CHANGELOG.txt` file or `README.txt file`, via the page source, or clues in the `robots.txt` file such as references to `/node`.

{% code title="Command" %}
```bash
curl -s http://drupal.acme.com | grep Drupal
```
{% endcode %}

{% code title="Output" %}
```bash
<meta name="Generator" content="Drupal 8 (https://www.drupal.org)" />
      <span>Powered by <a href="https://www.drupal.org">Drupal</a></span>
```
{% endcode %}

### Drupal Nodes

One way to identify Drupal CMS is through [nodes](https://www.drupal.org/docs/8/core/modules/node/about-nodes). Drupal indexes its content using nodes. A node can hold anything such as a blog post, poll, article, etc. The page URIs are usually of the form `/node/<nodeid>`.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

For example, the blog post above is found to be at `/node/1`. This representation is helpful in identifying a Drupal website when a custom theme is in use.

{% hint style="info" %}
## Note

Not every Drupal installation will look the same or display the login page or even allow users to access the login page from the internet.
{% endhint %}

***

## Users

Drupal supports three types of users by default:

1. `Administrator`: This user has complete control over the Drupal website.
2. `Authenticated User`: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
3. `Anonymous`: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.

***

## Manual Enumeration

Once we have discovered a Drupal instance, we can do a combination of manual and tool-based (automated) enumeration to uncover the version, installed plugins, and more.

Depending on the Drupal version and any hardening measures that have been put in place, we may need to try several ways to identify the version number. Newer installs of Drupal by default block access to the `CHANGELOG.txt` and `README.txt` files, so we may need to do further enumeration.

{% code title="Command" %}
```bash
curl -s http://drupal.acme.com/CHANGELOG.txt | grep -m2 ""
```
{% endcode %}

{% code title="Output" %}
```bash
Drupal 7.57, 2018-02-21
```
{% endcode %}

Here we have identified an older version of Drupal in use. Trying this against the latest Drupal version at the time of writing, we get a 404 response.

{% code title="Command" %}
```bash
curl -s http://drupal.acme.com/CHANGELOG.txt
```
{% endcode %}

{% code title="Output" overflow="wrap" %}
```html
<!DOCTYPE html><html><head><title>404 Not Found</title></head><body><h1>Not Found</h1><p>The requested URL "http://drupal.inlanefreight.local/CHANGELOG.txt" was not found on this server.</p></body></html>
```
{% endcode %}

There are several other things we could check in this instance to identify the version, such as using [droopescan.md](../../../toolbox/tooling/web-application-analysis/droopescan.md "mention").

***

## Automated Enumeration

### droopescan

Basic scan using [#droopescan](drupal-enumeration.md#droopescan "mention"):

{% code title="Command" %}
```bash
droopescan scan drupal -u http://drupal.acme.com
```
{% endcode %}

{% code title="Output" %}
```bash
[+] Plugins found:                                                              
    php http://drupal.acme.com/modules/php/
        http://drupal.acme.com/modules/php/LICENSE.txt

[+] No themes found.

[+] Possible version(s):
    8.9.0
    8.9.1

[+] Possible interesting urls found:
    Default admin - http://drupal.inlanefreight.local/user/login

[+] Scan finished (0:03:19.199526 elapsed)
```
{% endcode %}

This instance appears to be running version `8.9.1` of Drupal.

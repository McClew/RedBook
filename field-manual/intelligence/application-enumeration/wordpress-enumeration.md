# WordPress Enumeration

## Discovery & Footprinting

### robots.txt

A quick way to identify a WordPress site is by browsing to the `/robots.txt` file. A typical robots.txt on a WordPress installation may look like:

{% code title="/robots.txt" %}
```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://blog.acme.com/wp-sitemap.xml
```
{% endcode %}

### Page Source

Another quick way to identify a WordPress site is by looking at the page source. Use cURL and grep for '`WordPress`'.&#x20;

{% code title="Command:" %}
```bash
curl -s http://blog.acme.com | grep WordPress
```
{% endcode %}

{% code title="Output:" %}
```bash
<meta name="generator" content="WordPress 5.8" /
```
{% endcode %}

***

## Theme & Plugin Enumeration

### Page Source

Viewing the page source will give us hints to the theme in use, plugins installed, and even usernames if author names are published with posts. Spend time manually browsing the site and looking through the page source for each page, grepping for the `wp-content` directory, `themes` and `plugin`, and begin building a list of interesting data points.

#### Theme Enumeration

{% code title="Command" %}
```bash
curl -s http://blog.acme.com/ | grep themes
```
{% endcode %}

{% code title="Output" overflow="wrap" %}
```bash
<link rel='stylesheet' id='bootstrap-css'  href='http://blog.acme.com/wp-content/themes/business-gravity/assets/vendors/bootstrap/css/bootstrap.min.css' type='text/css' media='all' />
```
{% endcode %}

We can see from the output that the theme 'business-gravity' is being used in this example.

#### Plugin Enumeration

{% code title="Command" %}
```bash
curl -s http://blog.inlanefreight.local/ | grep plugins
```
{% endcode %}

{% code title="Output" overflow="wrap" %}
```bash
<link rel='stylesheet' i='contact-form-7-css'  href='http://blog.acme.com/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.acme.com/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.8' id='subscriber-js-js'></script>
<script type='text/javascript' src='http://blog.acme.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.8' id='validation-engine-en-js'></script>
<script type='text/javascript' src='http://blog.acme.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.8' id='validation-engine-js'></script>
<link rel='stylesheet' id='mm_frontend-css'  href='http://blog.acme.com/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.8' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.acme.com/wp-content/plugins/contact-form-7/includes/js/index.js?ver=5.4.2' id='contact-form-7-js'></script>
```
{% endcode %}

In this example we can see that the the plugins 'Contact Form 7' and 'mail-masta' are in use.

### Plugin Source Code

If directory listing is enabled, we may be able to navigate to a plugins directory. For example: `http://blog.acme.com/wp-content/plugins/mail-masta/`. Any files here are very often helpful in fingerprinting version numbers.

***

## Enumerating Users

### User Permissions

There are five types of users on a standard WordPress installation.

1. **Administrator:** This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code.
2. **Editor:** An editor can publish and manage posts, including the posts of other users.
3. **Author:** They can publish and manage their own posts.
4. **Contributor:** These users can write and manage their own posts but cannot publish them.
5. **Subscriber:** These are standard users who can browse posts and edit their profiles.

### Manual Username Enumeration

We can manually enumerate usernames using the default WordPress login portal at `/wp-login.php`.

A valid username and an invalid password results in the following message:

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

However, an invalid username returns that the user was not found.

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

This makes WordPress vulnerable to username enumeration, which can be used to obtain a list of potential usernames.

***

## Automated Enumeration

### WPScan

[wpscan.md](../../../toolbox/tooling/web-application-analysis/wpscan.md "mention") is an automated WordPress scanner and enumeration tool. It determines if the various themes and plugins used by a blog are outdated or vulnerable.

{% code title="Basic Command" %}
```bash
sudo wpscan --url http://blog.acme.com --enumerate --api-token <API_TOKEN>
```
{% endcode %}


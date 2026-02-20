# Joomla Enumeration

## Discovery & Footprinting

### robots.txt

{% code title="robots.txt" %}
```bash
# If the Joomla site is installed within a folder
# eg www.example.com/joomla/ then the robots.txt file
# MUST be moved to the site root
# eg www.example.com/robots.txt
# AND the joomla folder name MUST be prefixed to all of the
# paths.
# eg the Disallow rule for the /administrator/ folder MUST
# be changed to read
# Disallow: /joomla/administrator/
#
# For more information about the robots.txt standard, see:
# https://www.robotstxt.org/orig.html

User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```
{% endcode %}

### README.txt

We can fingerprint the Joomla version if the `README.txt` file is present.

{% code title="Command" %}
```bash
curl -s http://dev.acme.com/README.txt | head -n 5
```
{% endcode %}

{% code title="Output" %}
```bash
1- What is this?
	* This is a Joomla! installation/upgrade package to version 3.x
	* Joomla! Official site: https://www.joomla.org
	* Joomla! 3.9 version history - https://docs.joomla.org/Special:MyLanguage/Joomla_3.9_version_history
	* Detailed changes in the Changelog: https://github.com/joomla/joomla-cms/commits/staging
```
{% endcode %}

### joomla.xml

In certain Joomla installs, we may be able to fingerprint the version from JavaScript files in the `media/system/js/` directory or by browsing to `administrator/manifests/files/joomla.xml`.

{% code title="Command" %}
```bash
curl -s http://dev.acme.com/administrator/manifests/files/joomla.xml | xmllint --format -
```
{% endcode %}

{% code title="Output" %}
```bash
<?xml version="1.0" encoding="UTF-8"?>
<extension version="3.6" type="file" method="upgrade">
  <name>files_joomla</name>
  <author>Joomla! Project</author>
  <authorEmail>admin@joomla.org</authorEmail>
  <authorUrl>www.joomla.org</authorUrl>
  <copyright>(C) 2005 - 2019 Open Source Matters. All rights reserved</copyright>
  <license>GNU General Public License version 2 or later; see LICENSE.txt</license>
  <version>3.9.4</version>
  <creationDate>March 2019</creationDate>
  
 <SNIP>
```
{% endcode %}

The `cache.xml` file can help to give us the approximate version. It is located at `plugins/system/cache/cache.xml`.

### Page Source

We can often fingerprint Joomla by looking at the page source, which tells us that we are dealing with a Joomla site.

{% code title="Command" %}
```bash
curl -s http://dev.acme.com/ | grep Joomla
```
{% endcode %}

{% code title="Output" %}
```bash
<meta name="generator" content="Joomla! - Open Source Content Management" />
```
{% endcode %}

***

## Enumeration

### droopescan

[droopescan.md](../../../toolbox/tooling/web-application-analysis/droopescan.md "mention") is a plugin-based scanner that works for SilverStripe, WordPress, and Drupal with limited functionality for Joomla and Moodle.

{% code title="Basic scan" %}
```bash
droopescan scan joomla --url http://dev.acme.com/
```
{% endcode %}

### JoomlaScan

[joomlascan.md](../../../toolbox/tooling/web-application-analysis/joomlascan.md "mention") is a Python tool inspired by the now-defunct OWASP [joomscan](https://github.com/OWASP/joomscan) tool. JoomlaScan is a bit out-of-date and requires Python2.7 to run.

{% code title="Command" %}
```bash
python2.7 joomlascan.py -u http://dev.acme.dev
```
{% endcode %}

This tool can help us find accessible directories and files and may help with fingerprinting installed extensions.

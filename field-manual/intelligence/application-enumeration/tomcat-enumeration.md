# Tomcat Enumeration

[Apache Tomcat](https://tomcat.apache.org) is an open-source web server that hosts applications written in Java.

***

## Discovery & Footprinting

### Error Pages

Tomcat servers can be identified by the Server header in the HTTP response. If the server is operating behind a reverse proxy, requesting an invalid page should reveal the server and version.&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here we can see that Tomcat version `9.0.30` is in use.

{% hint style="info" %}
## Note

Custom error pages may be in use that do not leak this version information.
{% endhint %}

### /docs Page

Another method of detecting a Tomcat server and version is through the `/docs` page.

{% code title="Command" %}
```bash
curl -s http://app-dev.acme.com:8080/docs/ | grep Tomcat 
```
{% endcode %}

{% code title="Output" overflow="wrap" %}
```html
<html lang="en"><head><META http-equiv="Content-Type" content="text/html; charset=UTF-8"><link href="./images/docs-stylesheet.css" rel="stylesheet" type="text/css"><title>Apache Tomcat 9 (9.0.30) - Documentation Index</title><meta name="author" 
```
{% endcode %}

This is the default documentation page, which may not be removed by administrators. See the [#folder-structure](tomcat-enumeration.md#folder-structure "mention") section for more information.

***

## Enumeration

### Gobuster

After fingerprinting the Tomcat instance, unless it has a known vulnerability, we'll typically want to look for the `/manager` and the `/host-manager` pages. We can attempt to locate these with a tool such as **Gobuster** or just browse directly to them.

{% code title="Command" %}
```bash
gobuster dir -u http://web01.acme.com:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```
{% endcode %}

{% code title="Output" %}
```
<Snip>
===============================================================
2021/09/21 17:34:54 Starting gobuster
===============================================================
/docs (Status: 302)
/examples (Status: 302)
/manager (Status: 302)
Progress: 49959 / 87665 (56.99%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2021/09/21 17:44:29 Finished
===============================================================
```
{% endcode %}

We may be able to either log in to one of these using weak credentials such as `tomcat:tomcat`, `admin:admin`, etc. If these first few tries don't work, we can try a password brute force attack against the login page.

If we are successful in logging in, we can upload a [Web Application Resource or Web Application ARchive (WAR)](https://en.wikipedia.org/wiki/WAR_\(file_format\)) file containing a JSP web shell and obtain remote code execution on the Tomcat server.

***

## Folder Structure

Here is the general folder structure of a Tomcat installation.

```
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```

* The `bin` folder stores scripts and binaries needed to start and run a Tomcat server.
* The `conf` folder stores various configuration files used by Tomcat.
* The `tomcat-users.xml` file stores user credentials and their assigned roles.
* The `lib` folder holds the various JAR files needed for the correct functioning of Tomcat.
* The `logs` and `temp` folders store temporary log files.
* The `webapps` folder is the default webroot of Tomcat and hosts all the applications.
* The `work` folder acts as a cache and is used to store data during runtime.

### /webapps

Each folder inside `webapps` is expected to have the following structure:

```
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class  
```

* The most important file among these is `WEB-INF/web.xml`, which is known as the deployment descriptor. This file stores information about the routes used by the application and the classes handling these routes.
* All compiled classes used by the application should be stored in the `WEB-INF/classes` folder. These classes might contain important business logic as well as sensitive information. Any vulnerability in these files can lead to total compromise of the website.
* The `lib` folder stores the libraries needed by that particular application.
* The `jsp` folder stores [Jakarta Server Pages (JSP)](https://en.wikipedia.org/wiki/Jakarta_Server_Pages), formerly known as `JavaServer Pages`, which can be compared to PHP files on an Apache server.

### web.xml

Here’s an example `web.xml` file:

{% code title="web.xml" %}
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.acme.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```
{% endcode %}

The `web.xml` configuration above defines a new servlet named `AdminServlet` that is mapped to the class `com.inlanefreight.api.AdminServlet`. Java uses the dot notation to create package names, meaning the path on disk for the class defined above would be:

* `classes/com/inlanefreight/api/AdminServlet.class`

Next, a new servlet mapping is created to map requests to `/admin` with `AdminServlet`. This configuration will send any request received for `/admin` to the `AdminServlet.class` class for processing. The `web.xml` descriptor holds a lot of sensitive information and is an important file to check when leveraging a Local File Inclusion (LFI) vulnerability.

### tomcat-users.xml

The `tomcat-users.xml` file is used to allow or disallow access to the `/manager` and `host-manager` admin pages.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<SNIP>
  
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
<!--
  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

  Built-in Tomcat manager roles:
    - manager-gui    - allows access to the HTML GUI and the status pages
    - manager-script - allows access to the HTTP API and the status pages
    - manager-jmx    - allows access to the JMX proxy and the status pages
    - manager-status - allows access to the status pages only

  The users below are wrapped in a comment and are therefore ignored. If you
  wish to configure one or more of these users for use with the manager web
  application, do not forget to remove the <!.. ..> that surrounds them. You
  will also need to set the passwords to something appropriate.
-->

   
 <SNIP>
  
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />


</tomcat-users>
```

The file shows us what each of the roles `manager-gui`, `manager-script`, `manager-jmx`, and `manager-status` provide access to. In this example, we can see that a user `tomcat` with the password `tomcat` has the `manager-gui` role, and a second weak password `admin` is set for the user account `admin`.

***

## Common Gateway Interface

The CGI Servlet is a vital component of Apache Tomcat that enables web servers to communicate with external applications beyond the Tomcat JVM. These external applications are typically CGI scripts written in languages like Perl, Python, or Bash. The CGI Servlet receives requests from web browsers and forwards them to CGI scripts for processing.

### enableCmdLineArguments

The `enableCmdLineArguments` setting for Apache Tomcat's CGI Servlet controls whether command line arguments are created from the query string. If set to true, the CGI Servlet parses the query string and passes it to the CGI script as arguments. This feature can make CGI scripts more flexible and easier to write by allowing parameters to be passed to the script without using environment variables or standard input.

### Finding CGI Scripts

One way to uncover web server content is by utilising the [ffuf.md](../../../toolbox/tooling/web-application-analysis/ffuf.md "mention") web enumeration tool along with the `dirb common.txt` wordlist. Knowing that the default directory for CGI scripts is `/cgi`, we can use the URLs:

`http://<DOMAIN/IP>:8080/cgi/FUZZ.cmd`

`http://<DOMAIN/IP>:8080/cgi/FUZZ.bat`

{% code title="Command" %}
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
```
{% endcode %}

{% code title="Command" %}
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
```
{% endcode %}

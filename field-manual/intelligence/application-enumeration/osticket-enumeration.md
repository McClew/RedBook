# osTicket Enumeration

[osTicket](https://osticket.com) is an open-source support ticketing system. It can be compared to systems such as Jira, OTRS, Request Tracker, and Spiceworks. osTicket can integrate user inquiries from email, phone, and web-based forms into a web interface. osTicket is written in **PHP** and uses a **MySQL** backend.

It can be installed on **Windows** or **Linux**.

## Discovery & Footprinting

{% hint style="info" %}
## Nmap Scan

An Nmap scan will just show information about the webserver, such as Apache or IIS, and will not help us footprint the application.
{% endhint %}

### Web Request

Most osTicket installs will showcase the osTicket logo with the phrase `powered by` in front of it in the page's footer. The footer may also contain the words `Support Ticket System`.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### EyeWitness

From an [eyewitness.md](../../../toolbox/tooling/web-application-analysis/eyewitness.md "mention") scan we might find a screenshot of an osTicket instance. This shoudl also show a cookie named `OSTSESSID` was set when visiting the page.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

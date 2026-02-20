# Application Discovery & Fingerprinting

## Automated Tools

| Tool                                                                                       | Description                                                                                                                                                         |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [eyewitness.md](../../../toolbox/tooling/web-application-analysis/eyewitness.md "mention") | EyeWitness is designed to take screenshots of websites provide some server header info, and identify default credentials if known.                                  |
| [aquatone.md](../../../toolbox/tooling/web-application-analysis/aquatone.md "mention")     | Aquatone is a tool for visual inspection of websites across a large amount of hosts and is convenient for quickly gaining an overview of HTTP-based attack surface. |

***

## EyeWitness Basic Usage

For more information view [eyewitness.md](../../../toolbox/tooling/web-application-analysis/eyewitness.md "mention").

```bash
eyewitness --web -x nmap.xml -d eyewitness
```

* `--web`: HTTP Screenshot using Selenium.
* `-x`: Nmap XML or .Nessus file.
* `-d`: Directory name for the report output.

***

## Aquatone Basic Usage

For more information view [aquatone.md](../../../toolbox/tooling/web-application-analysis/aquatone.md "mention").

In the below example we are piping the [nmap](../../../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention") results XML file directly to Aquatone.

```bash
cat web_discovery.xml | ./aquatone -nmap
```

# OpenCTI Group-IB Connector

| Status            | Date       | Comment |
| ----------------- |------------| ------- |
| Filigran Verified | 2025-03-10 |    -    |

[![Python](https://img.shields.io/badge/python-v3.6.8+-blue?logo=python)](https://python.org/downloads/release/python-368/)
[![cyberintegrations](https://img.shields.io/badge/cyberintegrations-v0.6.6+-orange?)](https://github.com/cyberintegrations/releases/tag/0.6.6/)
[![OpenCTI](https://img.shields.io/badge/opencti-v6.3.1+-orange?)](https://github.com/OpenCTI-Platform/opencti/releases/tag/6.3.1)


<!--
General description of the connector
* What it does
* How it works
* Special requirements
* Use case description
* ...
-->

The OpenCTI Group-IB Connector is a standalone Python process that collect data from Threat Intelligence via API calls
and push it as STIX objects to OpenCTI server.

It  is a system for cyber-attack analysis and attribution, threat hunting, and network infrastructure protection 
based on data about adversary tactics, tools, and activities. TI combines unique data sources and experience in 
investigating high-tech crimes and responding to complex, multi-stage attacks worldwide. The system stores data 
on threat actors, domains, IPs, and infrastructure collected over the past 22 years, including those that criminals 
have attempted to take down.

To use the integration, please ensure that you have an active Threat Intelligence license to access the 
interface and that it covers the API endpoints you wish to reach. Documentation can be found here - https://tap.group-ib.com/hc/api?scope=integrations&q=en%2FIntegrations%2FCollections%20Info%2FCollections%20Details%2FCollections%20Details


## **Content**

- [OpenCTI Group-IB Connector](#opencti-group-ib-connector)
  - [**Content**](#content)
  - [**Installation**](#installation)
    - [Requirements](#requirements)
    - [Common environment variables](#common-environment-variables)
    - [OpenCTI environment variables](#opencti-environment-variables)
    - [Threat Intelligence API environment variables](#threat-intelligence-api-environment-variables)
    - [Threat Intelligence API environment variables](#threat-intelligence-api-environment-variables-1)
    - [Environment variables sample](#environment-variables-sample)
    - [Docker Deployment](#docker-deployment)
    - [Manual Deployment](#manual-deployment)
    - [Enable required collections](#enable-required-collections)
    - [Date format](#date-format)
    - [Notes](#notes)
  - [Extra settings](#extra-settings)
    - [Tags (in development)](#tags-in-development)
    - [Options](#options)
  - [Examples](#examples)
  - [**Task scheduling automation**](#task-scheduling-automation)
    - [Cron:](#cron)
    - [Task Scheduler:](#task-scheduler)
  - [Troubleshooting](#troubleshooting)
  - [FAQ](#faq)
    - [Debugging](#debugging)
    - [Additional information](#additional-information)



<br/>



## **Installation**

### Requirements

- Active Threat Intelligence license
- OpenCTI Platform >= 6.3.1


### Common environment variables

Configuration parameters are set in the .env or config.yml file, depending on the type of integration run:
- .env file is used when running with docker This file is included in the `.gitignore` (to avoid leaking sensitive data). 
- ~~config.yml is used when running locally without docker, as if you were running a regular python script.~~
/!\ Currently the connector cannot run with config.yml file, only with .env file because of ConnectorHelper instantiation and proxy settings code. /!\

All integration values are duplicated between .env and config.yml

Note that the values that follow can be grabbed within Python code using `self.helper.{PARAMETER}`, i. e., `self.helper.connector_name`.

Expected environment variables to be set in the  `docker-compose.yml` that describe the connector itself.
Most of the time, these values are NOT expected to be changed.

| Parameter                  | Mandatory  | Description                                                        |
|----------------------------|------------|--------------------------------------------------------------------|
| `CONNECTOR_NAME`           | Yes        | A connector name to be shown in OpenCTI.                           |
| `CONNECTOR_SCOPE`          | Yes        | Supported scope. E. g., `text/html`.                               |
| `CONNECTOR_ID`             | Yes        | A valid arbitrary `UUIDv4` that must be unique for this connector. |

However, there are other values which are expected to be configured by end users.
The following values are expected to be defined in the `.env` file.
Note that the `.env.sample` file can be used as a reference.

The ones that follow are connector's generic execution parameters expected to be added for export connectors.

| Parameter                      | Mandatory | Description                                                                                                                                                                   |
|--------------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CONNECTOR_LOG_LEVEL`          | Yes       | The log level for this connector, could be `debug`, `info`, `warn` or `error` (less verbose).                                                                                 |
| `CONNECTOR__DURATION_PERIOD`   | Yes       | The time unit is represented by a single character at the end of the string: d for days, h for hours, m for minutes, and s for seconds. e.g., 30s is 30 seconds. 1d is 1 day. |


### OpenCTI environment variables

Below are the parameters you'll need to set for OpenCTI:

| Parameter               | Mandatory | Description                                                                                                      |
|-------------------------|-----------|------------------------------------------------------------------------------------------------------------------|
| `OPENCTI_URL`           | Yes       | The URL of the OpenCTI platform. Note that final `/` should be avoided. Example value: `http://opencti:8080`     |
| `OPENCTI_TOKEN`         | Yes       | The default admin token configured in the OpenCTI platform parameters file.                                      |


### Threat Intelligence API environment variables

Below are the parameters you'll need to set for Threat Intelligence API:

| Parameter            |  Mandatory | Description                                    |
|----------------------|------------|-------------------------------------------------|
| `TI_API__URL`        |  Yes       | Threat Intelligence API URL.                   |
| `TI_API__USERNAME`   |  Yes       | Threat Intelligence Portal profile email.      |
| `TI_API__TOKEN`      |  Yes       | Threat Intelligence API Token.                 |


### Threat Intelligence API environment variables

Below are the parameters you'll need to set if you have proxy server (if necessary):

| Parameter                | Mandatory | Description     |
|--------------------------|-----------|-----------------|
| `PROXY_IP`               |  No       | Proxy IP.       |
| `PROXY_PORT`             |  No       | Proxy port.     |
| `PROXY_PROTOCOL`         |  No       | Proxy protocol. |
| `PROXY_USERNAME`         |  No       | Proxy username. |
| `PROXY_PASSWORD`         |  No       | Proxy password. |


### Environment variables sample

Environment variables explained above may be not that obvious. So we recommend you to start with the next steps:

Copy `.env.sample` to `.env`

```bash
cp .env.sample .env
```

Open file in any editor. Add unfilled mandatory variables (OPENCTI_TOKEN, TI_API_USERNAME, TI_API_TOKEN) and save changes

```bash
nano .env
```


### Docker Deployment

Before building the Docker container, you need to set the version of pycti in `requirements.txt` 
equal to whatever version of OpenCTI you're running. Example, `pycti==6.2.0`. If you don't, it will take 
the latest version, but sometimes the OpenCTI SDK fails to initialize.

Build a Docker Image using the provided `Dockerfile`.

```bash
docker compose up -d
# -d for detached
```


### Manual Deployment

Install the required python dependencies (preferably in a virtual environment):

```bash
pip3 install -r requirements.txt
```

Then, start the connector from `src`:

```bash
python3 main.py
```



<br/>


### Enable required collections

The parameter ```default_date``` is used for initial start only. 
After the download process begins it will not be used anymore.
After the first startup, the sequpdate field starts to be used instead of default_date, and its values for all collections are stored in the state of OpenCTI itself. 
If you need a fresh startup based on ``default_date`` then:
1. Stop the integration
2. Clear the state in OpenCTI that relates to our integration
3. Set a new date in ``default_date``. 
4. Restart the integration

To start download process for any collection:
  1. You need to enable it first - Set ```enable``` parameter to ```true```.
  2. Set ```default_date``` parameter in single quotes, if needed or leave it as ```null```. 

Example configuration in config.yml:
```yaml

collections:
    attacks/ddos:
        default_date: '2021-08-01'
        enable: true
    attacks/phishing_group:
        default_date: '2021-08-01'
        enable: true
    ioc/common:
        default_date: '2021-08-01'
        enable: true
...
```
Example configuration in .env:
```
TI_API__COLLECTIONS__APT_THREAT__DEFAULT_DATE='2021-08-01'
TI_API__COLLECTIONS__APT_THREAT__ENABLE=false
TI_API__COLLECTIONS__APT_THREAT__LOCAL_CUSTOM_TAG=null
TI_API__COLLECTIONS__APT_THREAT__TTL=90
...
```

### Date format

Default date format.

```'YYYY-MM-DD'```

### Notes

*Note*: To use only IOCs (for example - Firewall rules), enable next collection: ```ioc/common```.

*Note*: The ```ioc/common``` collection contains IoC only and based on ```malware/cnc```, 
```hi/threat```, ```apt/threat```, ```hi/threat_actor```, ```apt/threat_actor``` collections. 

*Note*: ```attacks/deface```, ```attacks/ddos```, ```attacks/phishing_group```, ```suspicious_ip/open_proxy```, 
```suspicious_ip/socks_proxy```, ```suspicious_ip/open_proxy```, ```suspicious_ip/tor_node```, 
```suspicious_ip/vpn```, ```suspicious_ip/scanner``` - are very large collections,
and it is recommended to specify in the default_date field: 1-3 days ago. 
Learn more about each collection 
[here](https://tap.group-ib.com/hc/api?scope=integrations&q=en%2FIntegrations%2FDetailed%20collections%20info%2FDetailed%20collections%20info).


<br/>



## Extra settings


### Tags (in development)

To mark events with custom tags for each collection you can use `local_custom_tag` 
parameter in `endpoints_config.yaml` file.

```yaml

collections:
    attacks/ddos:
        default_date: '2021-08-01'
        enable: true
        local_custom_tag: 'my_ddos_tag'
    attacks/phishing_group:
        default_date: '2021-08-01'
        enable: true
        local_custom_tag: 'my_phishing_tag'
...
```

### Options

An Intrusion Set can be created above a Threat Actor. 
All related objects will be linked to this Intrusion Set along with Threat Actor. 
If you need such reorganisation of all relations, set 
`intrusion_set_instead_of_threat_actor` parameter to `true` in `endpoints_config.yaml` file for that purpose. 
We recommend clearing the data before this and start the download process again for the clear view.

```yaml

extra_settings:
  intrusion_set_instead_of_threat_actor: true
...
```

To ignore DDoS events without malware payload set `ignore_non_malware_ddos` parameter to `true`
in `endpoints_config.yaml` file.

```yaml

extra_settings:
  ignore_non_malware_ddos: true
...
```

To ignore Threat events without indicators set `ignore_non_indicator_threats` parameter to `true`
in `endpoints_config.yaml` file.

```yaml

extra_settings:
  ignore_non_indicator_threats: true
...
```




<br/>



## Examples

Threat Reports

![Reports](./__docs__/media/reports.png)

Threat Report with TI direct links 

![Report](./__docs__/media/report.png)

Threat Report `Knowledge` tab graph

![Report graph](./__docs__/media/report_graph.png)

Indicators based on Observables

![Indicators](./__docs__/media/indicators.png)

Threat Report Actors

![Threat actors](./__docs__/media/threat_actors.png)

Threat Report Actor with related objects

![Threat actor](./__docs__/media/threat_actor.png)

Threat Report Actor TTP

![Threat actor TTP](./__docs__/media/threat_actor_ttp.png)

The way how relations names organized

![mapping relationships](./__docs__/media/mapping-relationships.png)


<br/>



## **Task scheduling automation**

Use operating system (cron or task scheduler) opportunities to automate daily 
```main.py``` script execution.
You can use [Cron](https://crontab.guru/every-midnight) (for Unix-based systems) or Task Scheduler (for Windows). 
The User must have appropriate rights.

### Cron:

-   Open file **/etc/crontab**
-   Set the following job:

```

0 0 * * * cd %<Path_to_script_directory>% && %<Path_to_python>%/python %<Path_to_script>%/main.py  
```

-   This job will start polling daily at midnight

### Task Scheduler:

-   Create a new file **poller.bat** with the following content:  

```

“%<Path_to_python.exe>%” “%<Path_to_python_script>%\main.py”  
pause
```

-   Go to **Control Panel** → Administrative Tools → Task Scheduler  
-   Choose the **Create Task** option. 
-   Fill in the task name and description. 
-   In the **Triggers** tab create a new daily trigger and set it to be repeated each 12 hours for an infinite amount of days. 
-   In the **Actions** tab add a new action and select the .bat file created on the first step. 
-   Click the **Ok** button.



<br/>



## Troubleshooting

1. If you encounter any problems, please retrieve logs from the **log** folder and attach them to 
[Email](mailto:integration@group-ib.com) 
or 
[Service Desk](https://tap.group-ib.com/service_desk) 
ticket. Also, please provide your TI portal email address and public IP address of integration app instance 
(docker container IP / virtual machine IP).
If the **log** folder doesn't exist, please collect logs from console output or app container output.
    
    - Console output (run app with redirecting output to file `app_logs.log`)
        ```bash
        python3 main.py > app_logs.log
        ```
    - App container output (retrieve container id and use `docker logs` command with output redirect to `app_logs.log` file, for last hour)
        ```bash
        docker ps -a
        docker logs --since=1h <container_id> > app_logs.log
        # example
        docker ps -a
        # CONTAINER ID   IMAGE       COMMAND    CREATED   STATUS  PORTS                                                                  NAMES
        # b78e4ebf809d   ...         ...        ...       ...     ...
        docker logs --since=3h b78e4ebf809d > app_logs.log
        ```
	
2. If you have problems with proxy configuration, attach the proxy environment by executing this command: 
```printenv | grep proxy```

3. If you encounter any problem when activate a collection and 403 status response is raised:
```ConnectionException: Status code: 403. Message: Something is wrong with your account, please, contact us. The issue can be related to Access list, Wrong API key or Wrong username.", "taskName": null}```

Please ensure that:
- **IP is in Access List**: Provide your public IP addresses, for GroupIB to add them to the API access list.
- **Generate API Key if expired**: Log in to your account, navigate to your profile, and generate an API_KEY. Be sure to save this key, as it will be required for API access. For API authorization, use your email and the generated API key instead of your portal password. 

## FAQ

1. Where I can find reports from last threats?

     They are separated similarly to TI interface. 
     **hi/threat** stands for Cybercriminals and **apt/threat** stands for Nation-State.

2. What tags we are using for our events?

    Besides tags, that you can set in the configuration, we are using:
    - collection name (but with spaces instead of "_" or "/")
    - admiralty codes and TLP, where applicable
    - for **osi/vulnerability** we add affected products as tags
    - for **hi/threat** and **apt/threat** we add "Tailored" tag, similar to the TI portal

3. Why app raise an access deni error in logs?

    Please check that OpenCTI user has correct rights and access. 
    Also, please check if you install integration app separate from the OpenCTI server virtual machine/docker container.
    OpenCTI server is a separate app which control its folders rights.


### Debugging

The connector can be debugged by setting the appropriate log level.
Note that logging messages can be added using `self.helper.log_{LOG_LEVEL}("Sample message")`, i. e., `self.helper.log_error("An error message")`.

<!-- Any additional information to help future users debug and report detailed issues concerning this connector -->

### Additional information

<!--
Any additional information about this connector
* What information is ingested/updated/changed
* What should the user take into account when using this connector
* ...
-->

If you face any errors with OpenCTI server images, remove existing docker images. 
Warning, all your docker images will be deleted:

```bash
rm -rf /var/lib/docker/*
```
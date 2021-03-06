Automatic Application Monitoring - v2
===
This Zabbix "module" provides automated Application Component monitoring by utilizing the Low-Level Discovery functionality of Zabbix.

It automatically creates items and triggers based on a list of components provided by the application.

Background details: https://enrise.com/2015/10/automating-application-component-monitoring/

## Requirements for your application
In your application you should create two endpoints which should be available as a page which responds with JSON.

### Key endpoint
This endpoint provides all available keys to Zabbix, which is used in the auto-discovery of components.
Based on the content, items and triggers will be created from the prototype.

This endpoint should return a JSON array of available keys like this:
```json
{
    "Unclassified Component": {
        "severity": "unclassified"
    },
    "Information Component": {
        "severity": "information"
    },
    "Warning Component": {
        "severity": "warning"
    },
    "Average Component": {
        "severity": "average"
    },
    "High Component": {
        "severity": "high"
    },
    "Disaster Component": {
        "severity": "disaster"
    }
}
```

### Status endpoint
This endpoint provides the generated items with data, on which the triggers will be evaluated.

This endpoint should return the status in JSON for all available components:
```json
{
    "Unclassified Component": {
        "statusCode": 0,
        "severity": "unclassified"
    },
    "Information Component": {
        "statusCode": 0,
        "severity": "information"
    },
    "Warning Component": {
        "statusCode": 0,
        "severity": "warning"
    },
    "Average Component": {
        "statusCode": 0,
        "severity": "average"
    },
    "High Component": {
        "statusCode": 0,
        "severity": "high"
    },
    "Disaster Component": {
        "statusCode": 0,
        "severity": "disaster"
    }
}
```
## Installation in Zabbix

* Install the dependencies for AAMv2:
```shell
$ sudo pip install -r requirements.txt
```
If pip is not present on your system:
```shell
$ sudo easy_install pip
```

* Add `aamv2.py` to your external scripts (default: `/usr/lib/zabbix/externalscripts/`) folder on your Zabbix Server and ensure it is executable.

* Under `Configuration => Templates` in the Zabbix webinterface import `zbx_aamv2_template.xml`
* To the host you want to monitor, add the following macro's:

| Macro                  | Value                                 |
|------------------------|---------------------------------------|
| {$AAM_KEY_ENDPOINT}    | http://your-application/appmon/keys   |
| {$AAM_STATUS_ENDPOINT} | http://your-application/appmon/status |

*The URL's should point towards the endpoints you have created. They can be routes, individual pages, static files or queryable with HTTP-Parameters depending on your application as long as the output matches what the module requires.*

* Optionally you can override the default values for some querying periods:

| Macro                      | Default value  | Usage                                                                                              |
|----------------------------|----------------|----------------------------------------------------------------------------------------------------|
| ${AAM_THRESHOLD_COMPONENT} |       5m       | Threshold for usage in `min()` on the discovered components                                        |
| {$AAM_PERIOD_STATUS}       |       5m       | Used to override the template's trigger in case the period has been modified on the inherited item |
| {$AAM_PERIOD_DISCOVERY}    |       30m      | Used to override the template's trigger in case the period has been modified on the inherited item |

* Assign the template `Template Automatic Application Monitoring v2` to the host you want to monitor the application of. If your host has multiple applications its recommended to create a dummy host in Zabbix for each application on the server.
* After some time it will automatically create the items and trigger for status reporting

## Demo app
The folder `demoapp` contains a [Lumen](http://lumen.laravel.com/) PHP application which showcases the output it can generate. The most interesting part of the code is the `app/Http/Controllers/StatusController.php`.

This app serves as a demo on how to implement it. It is in no way a requirement to be able to use the Zabbix module. As long as your application produces the correct JSON output you can use any programming language or framework.

## Contributing
Contributions to this module and its implementation are welcome!

### License
The MIT License (MIT). Please see the [license file](LICENSE) for more information.

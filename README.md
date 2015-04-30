TimberWinR
==========
A Native Windows to Redis/Elasticsearch Logstash Agent which runs as a service.

## Why have TimberWinR?
TimberWinR is a native .NET implementation utilizing Microsoft's [LogParser](http://technet.microsoft.com/en-us/scriptcenter/dd919274.aspx).  This means
no JVM/JRuby is required, and LogParser does all the heavy lifting.  TimberWinR collects
the data from LogParser and ships it to Logstash via Redis (or can ship direcly to Elasticsearch)

## Release Notes

[View Version History](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/ReleaseNotes.md)

## Basics
TimberWinR uses a configuration file to control how the logs are collected, filtered and shipped off.  
These are broken down into:
 1. Inputs  (Collect data from different sources)
 2. Filters (Are applied to all Inputs)
 3. Outputs (Redis, Elasticsearch or Stdout)

### Support ###
Please use the TimberWinR Google Group for discussion and support:

https://groups.google.com/forum/#!forum/timberwinr

Latest Build:

![alt tag](https://ci.appveyor.com/api/projects/status/qi5x2lg153h1tim6/branch/master?svg=true)


## Inputs
The current supported Input format sources are:
 1. [Logs](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/Logs.md) (Files, a.k.a Tailing a file)
 2. [Tcp](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/TcpInput.md) (listens on TCP port for JSON messages)
 3. [IISW3C](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/IISW3CInput.md)(Internet Information Services W3C Format)
 4. [WindowsEvents](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/WindowsEvents.md) (Windows Event Viewer)
 5. [Stdin](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/StdinInput.md) (Standard Input for Debugging)
 6. [W3C](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/W3CInput.md)(Internet Information Services W3C Advanced/Custom Format)
 7. [Udp](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/UdpInput.md) (listens for UDP on port for JSON messages)
 8. [TailFiles](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/TailFiles.md) (Tails log files efficiently)
 8. [Generator](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/Generator.md) (Generate logs for testing *New*)

## Codecs
The current list of supported codecs are:
 1. [Multiline](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/Codec.md)

## Filters
The current list of supported filters are:
 1. [Grok](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/GrokFilter.md)
 2. [Mutate](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/MutateFilter.md)
 3. [Date](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/DateFilter.md)
 4. [Json](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/JsonFilter.md)
 5. [GeoIP](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/GeoIPFilter.md)

Note that there are now two syntaxes for filters, array and single, if you want more than one filter of the same
type then you must use the array tag instead of the singular tag. i.e:

A single Json filter using the single tag (this is only provided as a convienience, the array syntax is preferred)
```json
"Filters": [          
    {
        "json": {
            "type": "Win32-FileLog",
            "condition": "\"[logSource]\" == \"dev\"",
            "source": "Text",
            "add_field": [
                "_index",
                "dev-%{yyyy.MM.dd}"
            ]
        }
    }
]
```
Multiple Json filters must use the jsonFilters and array syntax, also mutateFilters, grokFilters, dateFilters, geoipFilters.
```json
"Filters": [
    {
        "jsonFilters": [
            {
                "type": "Win32-FileLog",
                "condition": "\"[logSource]\" == \"dev\"",
                "source": "Text",
                "add_field": [
                    "_index",
                    "dev-%{yyyy.MM.dd}"
                ]
            },
            {
                "type": "Win32-FileLog",
                "condition": "\"[logSource]\" == \"sta\"",
                "source": "Text",
                "add_field": [
                    "_index",
                    "sta-%{yyyy.MM.dd}"
                ]
            }
        ]
    }
]
```

## JSON
Since TimberWinR only ships to Redis and Elasticsearch, the format generated by TimberWinR is JSON.  All fields referenced by TimberWinR can be
represented as a JSON Property or Array.

## Outputs
1. [Redis](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/RedisOutput.md)
2. [Elasticsearch](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/ElasticsearchOutput.md)
3. [Stdout](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/StdoutOutput.md)
4. [File](https://github.com/Cimpress-MCP/TimberWinR/blob/master/TimberWinR/mdocs/FileOutput.md)

## Sample Configuration
TimberWinR reads a JSON configuration file, an example file is shown here:
```json
{
"TimberWinR": {
    "Inputs": {
        "WindowsEvents": [
            {
                "source": "System,Application",
                "binaryFormat": "PRINT",
                "resolveSIDS": true
            }
        ]
    },
    "Filters": [          
        {
            "grok": {
                "condition": "\"[type]\" == \"Win32-Eventlog\"",
                "match": [
                    "Message",
                    ""
                ],                   
                "remove_field": [
                    "ComputerName"                   
                ]
            }
        }
    ],
    "Outputs": {
        "Redis": [
            { 
                "_comment": "Shuffle these hosts",
                "host": [
                   "server1.host.com", 
                   "server2.host.com"
                ]
            }
        ]
    }
}
```
This configuration:
 1. Inputs: Events from the Windows Event Logs (System, Application)
 2. Filters: Removes the ComputerName field
 3. Sends the event to Redis services (server1.host.com, server2.host.com) in a shuffling manner (balanced).

## Installation from Source
If installing from source, you must first install LogParser, then install TimberWinR.   Install LogParser from here:

[Install LogParser](http://www.microsoft.com/en-us/download/details.aspx?id=24659) from Microsoft.

After installing, follow the remaining directions here.   If you install from [Chocolatey](https://chocolatey.org/packages/TimberWinR) then LogParser will automatically
be installed first.

## Running Interactively
You can run TimberWinR interactively when you are developing your JSON config file, to do so use the
following options:
```
TimberWinR.ServiceHost.exe -configFile:myconfig.json -logLevel:Debug
```

## Automatic Installation via Chocolatey (embedded)

[TimbeWinR Chocolatey](https://chocolatey.org/packages/TimberWinR)

![alt tag](https://raw.github.com/Cimpress-MCP/TimberWinR/master/chocolatey.png)

## Manual Installation as a Windows Service
TimberWinR uses [TopShelf](http://topshelf-project.com/) to install as a service, so all the documentation
for installing and configuring the service is show here [TopShelf Doc](http://docs.topshelf-project.com/en/latest/)

Specifically the command line options are listed here in [Topshelf Command-Line Reference](http://docs.topshelf-project.com/en/latest/overview/commandline.html) guide.

Install and set to Automatically Start the service:
```
; Install Service (will autostart on reboot)
TimberWinR.ServiceHost.exe install --autostart
; Start the Service
TimberWinR.ServiceHost.exe start
```

To Start/Stop the Service from the Command Line
```
TimberWinR.ServiceHost.exe start
TimberWinR.ServiceHost.exe stop
```

Alternatively you can use the Services Control Panel.
### Usage
```
TimberWinR.ServiceHost.exe [options]

Options:
-logDir:           Specifies the directory where TimberWinR will write its log file TimberWinR.txt
                   Default is -logDir:"C:\logs"
-logLevel:         Specifies the logging level for TimberWinR
                   Legal Values: Trace|Debug|Info|Warn|Error|Fatal|Off
                   Default is -logDir:Info
-configFile:       Specifies the path to the JSON config file, or directory which contains .json file(s).
                   Default is -configFile:default.json
-diagnosticPort:   Specifies the diagnostic port which can be used to get a health check of the service.
                   Default Port is 5141, A value of 0 will disable it.  Open a browser
                      http://localhost:5141
```
#### -configFile
This may be a single .json file or a directory containing .json file(s).  If it is a directory, all
files will be read and processed, the order in which the files will be processed will match the alphabetical
order on disk.

### Quickstart Guide
If you really just want to try it out, grab the binary distribution, extract the .zip file
into a directory, e.g.  C:\TimberWinR

Grab the [JSON example file](https://raw.githubusercontent.com/Cimpress-MCP/TimberWinR/master/TimberWinR.ServiceHost/default.json) and place it into C:\TimberWinR\default.json. 
Edit the default.json file and change the Redis instance to match yours, replace 'tstlexiceapp006.mycompany.svc' with the IP or DNS name
of the machine running redis.  Fire up the collector, enable the verbose debugging to see some Windows Events.

```
TimberWinR.ServiceHost.Exe -configFile:default.json -logLevel:Debug
```

You should see 


To run it as a service
```
TimberWinR.ServiceHost.exe install --autostart
TimberWinR.ServiceHost.exe start
```

### Builds ###
TimberWinR is distributed as an installable package via Chocolatey, and it is dependent on 

[TimbeWinR Chocolatey](https://chocolatey.org/packages/TimberWinR)

![alt tag](https://raw.github.com/Cimpress-MCP/TimberWinR/master/chocolatey.png)

#### Notes ####
After you install it via Chocolatey, you will have a folder:
```
C:\Program Files (x86)\TimberWinR
```
TimberWinR will be configured to read the file default.json located in the folder above.  You can edit this file
to customize your installation.

Use these commands to Stop/Start the service.

```
sc stop TimberWinR ; stop the service
sc start TimberWinR; start the service
```



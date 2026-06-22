# shellyPDU
A Power Distribution Unit based on Shelly Pro PM 4, capable of toggling each of the 4 different outputs to remotely power cycle a server.

(C) Jonas Bjurel et Al.

License: Apache 2 

## Description:
The purpose of the shellyPDU script is to remotely power-cycle equipment such as servers, switches, etc. Even if connectivity is lost due to the power-cycle the shellyPDU script ensures that the power is resumed after *reset_time_setting* seconds.
A power cyckle event can be triggered from the Shelly local-/cloud App by simply shutting off the wanted switch/relay, or by issuing a webhook: *http://<ShellyURL>/script/<script_id>/pdu?toggleBranch:<branch_id>*. 


## Script configuration (persistant)
This script's behaviour depends on script configuration settings with default values as defined in the
script under "default settings...". The default script configurations are persistantly written to the
shelly KVS (Key Value Store) at the first startup of the script, or after a factory reset of the script/
or the device. The default settings can be changed through the provided Shelly KVS HTTP APIs,
or alternatively setting the KVS store from the shelly local- or cloud- web-page.<br>
CAUTION: The shelly KVS store is using a storage with limited number of writes (~100 K), limit the number
of programatically initiated re-configurations to ensure adequate life-time of the device.

Following script setting/HTTP APIs are supported (GET):

**Hostname:**<br>
*http://"ShellyURL"/rpc/KVS.Set?key="hostname_setting"&value=\<hostname>*<br>
Sets the hostname of the Shelly device.

**Script scaning interval:**<br>
*http:<//"ShellyURL">/rpc/KVS.Set?key="scan_interval"&value=<scan_interval>*<br>
Sets the scripts scanning interval which in practice sets the granularity of the "resume power" timers.


**reset_time_setting**<br>
*http://"ShellyURL"/rpc/KVS.Set?key="reset_time_setting"&value=\<reset_time_setting_sec>*<br>

**Log-level:**<br>
*http://<"ShellyURL">/rpc/KVS.Set?key="log_level_setting"&value=<"LOG_CRITICAL" | "LOG_ERROR" | "LOG_WARN" | "LOG_INFO" | "LOG_VERBOSE">*<br>
Sets log level.

## Script interaction APIs (non persistant)
This PDU script provides non persistant run-time HTTP APIs that enables interaction with the PDU script and that retreives PDU information.

### Setting non persistant properties through HTTP APIs

**Factory reset:**<br>
*http://"ShellyURL"/script/\<scriptId>/pdu?factory_reset_to_default*<br>
Resets and restarts the PDU script to factory default. Default settings as defined in the script will persistantly be applied to the KVS store and any custom configurations needs to be applied again as described in
the "Script configuration (persistant)" section above. This method does not reset the shelly device as a whole to factory default, but only the PDU script it self.

Response body: Undefined

**Restart:**<br>
*http://"ShellyURL"/script/\<scriptId>/pdu?restart*<br>
Restarts the PDU script, all persistant configurations are retained - but the the internal state machine is re-started, meaning that all branches are powered up and normal operation is resumed.

Response body: Undefined

### Requesting status through HTTP APIs

**measurePerformanceMetrics:**<br>
*http://"ShellyURL"/script/\<scriptId>/pdu?measurePerformanceMetrics*<br>
Starts an asynchronous measurement of platform and PDU script system performance for which the results can later be picked-up
by a following "getPerformanceMetricMeasurements" call (see below).

Response body: An informative text string indicating the URL for which the result can be fetched.

**getPerformanceMetricMeasurements:**<br>
*http://"ShellyURL"/script/\<scriptId>/pdu?getPerformanceMetricMeasurements*<br>
Fetches the results from a previous "measurePerformanceMetrics" asynchronous request.

Response body: A JSON object<br>
{performanceMetrics: {metricsUpdated: <true|false>,
                      upTime:<value[s],
                      cpuLoadPerc:<value [%]>,
                      memUsed:<value [B]>,
                      memUsedPerc:<value [%]>,
                      memFree:<value [B]>,
                      memFreePerc:<value [%]>,
                      memHighWatermark:<value [B]>,
                      memHighWatermarkPerc:<value [%]>,
                      totalMem:<value [B]>,
                      overRuns:<value>,
                      callQueueLength:<value>,
                      callQueueLengthHighWaterMark:<value>}}

* **metricsUpdated:** Indicates if this was the first "getPerformanceMetricMeasurements" call after a successful "measurePerformanceMetrics" call, indicating weather the measurements are fresh/latest or stale from a previous measurement.
* **upTime:** Script uptime in seconds.
* **cpuLoadPerc:** Script portion of device CPU loading in %.
* **memUsed:** Script memory usage in Bytes.
* **memUsedPerc:** Script memory usage in %.
* **memFree:** Device memory free in Bytes.
* **memFreePerc:** Device memory free in %.
* **memHighWatermark:** Maximum ever used memory by Script in Bytes.
* **memHighWatermarkPerc:** Maximum ever used memory by Script in %.
* **totalMem:** Total available device memory in Bytes.
* **overRuns:** Total amount of accumulated overruns, I.e. when a time bound scan of currents, actions, etc cannot be expedited becaus the previous scan has not finished.
* **callQueueLength:** Current Shelly.call serilization queue length.
* **callQueueLengthHighWaterMark:** Longest Shelly.call serilization queue length since last reboot.
   
**Get PDU status:**<br>
*http://"ShellyURL"/script/\<scriptId>/pdu?getPduStatus*<br>
Provides current PDU status.

Response body: A JSON object:<br>
{"pduStatus":
{"totalCurrent":<total current>,"totalPower":<total power>,"voltage":<system voltage>,"freq":<system AC frequency>,<br>
"branches":{"0":{"current":<branch current>,"power":<branch power>,"isOn":<true | false>},<br>
            "1": ....<br>
            }}}<br>

**Power-cycle PDU branch:**<br>
*http://"ShellyURL"/script/\<scriptId>/pdu?toggleBranch:<branch id>*<br>
Power-cycles the provided branch.

Response body: An order confirmation.<br>

## Watchdog script
To monitor the shellyPDU script, úse the watchdog script provided with the shellyShedder repo: https://github.com/jonasbjurel/shellyShedder

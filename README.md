# telegraf nvidia-smi plugin on steroid

You like it? You use it? It saved you a couple of hours? Hey 👋👋👋 [You can now buy me a coffee](https://www.buymeacoffee.com/xrrxrr)! ☕️ 

There's already an[ NVIDIA SMI Telegraf Input Plugin](https://www.influxdata.com/integration/nvidia-smi/) but there's a limited number of fields and for some mysertious reasons it relies on some go code to invoke and parse nvidia-smi output.

nvidia-smi is capable of outputing some XML but I don't like XML processing with telegraf so I use xq, a python based tool, to transform the XML to JSON.

I siumply use the [exec telegraf input plugin](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/exec) to send that to an influxdb bucket.

I made the choice to keep all fields (except gpu_processes) in the example configuration but you can use *excluded_keys*, *included_keys* to refine the volume of data flowing to the db.

Enjoy !

## limitations & drawback

Units are included in the XML Output and therefore everything is processed as strings... This requires a bit more work on the grafana dashboard requests, and it also have an impact on Influxdb storage. Might be a future improvement to remove the units so that telegraf can process metrics as numerals, or not...


##Installation

I'm not going through telegraf output configuration/installation refer to the [Telegraf Documentation](https://docs.influxdata.com/telegraf/v1.26) for that.


* Make sure you install yq for the python telegraf is using

`sudo /usr/bin/python3 -m pip install yq`

* Copy yq to /usr/local/bin (or whatever location you prefer that is reachable by the telegraf user

* Verify that the following command is outputing json:

```
>/usr/bin/nvidia-smi -q -x | /usr/local/bin/xq .

{
  "nvidia_smi_log": {
    "timestamp": "Sat Mar 18 21:32:40 2023",
    "driver_version": "525.85.07",
    "cuda_version": "Not Found",
    "vgpu_driver_capability": {
      "heterogenous_multivGPU": "Supported"
    },
    "attached_gpus": "1",
    "gpu": {
      "@id": "00000000:84:00.0",
      "product_name": "Quadro P6000",
      "product_brand": "Tesla",
      "product_architecture": "Pascal",
      "display_mode": "Enabled",
      
      ...
```


##Telegraf configuration - Example
```
[[inputs.exec]]
  ## Commands array
  commands = ["/usr/bin/env bash -c '/usr/bin/nvidia-smi -q -x | /usr/local/bin/xq .'"]
  data_format = "json_v2"

[[inputs.exec.json_v2]]
  measurement_name = "nvidia-smi"
  timestamp_key= "nvidia_smi_log.timestamp"
    
[[inputs.exec.json_v2.object]]
  path = "nvidia_smi_log"

  excluded_keys = ["gpu_processes"]

[[inputs.exec.json_v2.object]]
  path = "nvidia_smi_log"
```



You like it? You use it? It saved you a couple of hours? 
Hey 👋👋👋 [You can now buy me a coffee](https://www.buymeacoffee.com/xrrxrr)! ☕️ 
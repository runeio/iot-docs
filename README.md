# Rune IOT Documentation

## Device Uploads

### Upload Paths
 All uploads are posted to:

| Environment | Path           |
| :---------- | :------------- |
| DEV |  `https://s3.amazonaws.com/rune-upload-d37c4bd756fde5287fa52b12ba9b40c9/<thing-name>/<epochtime>/<upload-type>` |
| STG | _ |
| PRD | _ |

### Upload Types

* **firstboot.gz**. On initial boot and before NTP is configured, the device will post it's initial configuration
* **fingerprint.gz**. Includes device hardware configuration as well as current OS snapshot (memory, cpu, swap, uptime, time)
* **hids.gz**. Host IDS payload, e.g. Snort.
* **fw.gz**. Firewall logs, .e.g. Netfilter or IPtables
* **proxy.gz**. Proxy logs, e.g. Squid
* **av.gz**. Antivirus logs, e.g. ClamAV
* **dns.gz**. DNS cache proxy logs.
* **flow.gz**. Network flow logs, e.g. Nmap

## MQTT Topics

Devices will subcribe and publish to the following list of MQTT topics:

| Topic     | Pub/Sub     | Purpose |
| :-------- | :---------- | :------ |
| `device/upload` | Publish | Send payload S3 URL after upload |
| `$aws/things/<thing-name>/shadow/get` | Publish | Send an empty message `""` to request desired state document |
| `$aws/things/<thing-name>/shadow/get/accepted` | Subscribe | Query to get desired state document |

## Device Telemetry

The AWS IOT Shadow Device service allows us to maintain the state of each thing/device on the backend. It provides a pub/sub model to maintaining state. The device checks the shadow to know when to update it's firmware and when to push data to AWS.

The JSON document below is an example of a typical Rune shadow device:

```
{
    "state": {
        "desired": {
            "firmware": {
                "builddate" : "20160328",
                "path" : "https://https://s3.amazonaws.com/rune-dev-firmware/20160328/firmware.gz",
                "checksum" : "76b547664f9bf560852503a3abb74173"
                },
            "fingerprint" : {
                "interval" : 300
            },
            "hids" : {
                "signature" : "abc4567890",
                "path" : "https://https://s3.amazonaws.com/rune-dev-hids/abc4567890/signature.gz",
                "checksum" : "52b547664f9bf560852503a3abb1730",
                "interval" : 60
            },
            "fw" : {
                "interval" : 120
            },
            "proxy" : {
                "interval" : 30
            },
            "av" : {
                "signature" : "1234567890",
                "path" : "https://https://s3.amazonaws.com/rune-dev-av/1234567890/signature.gz",
                "checksum" : "96b547664f9bf560852503a3abb7279",
                "interval" : 300
            },
            "dns" : {
                "interval" : 300
            },
            "flow" : {
                "interval" : 300
            }
        }
    }
}
```

In the above example the latest firmware is published as 20160328 together with the path and checksum of the binary in order to verify and install it. Next there's a separate interval period set for each upload type. This model provides not only flexbility to control which uploads happen when but also to manage bandwidth consumed by the device to upload these various data types.

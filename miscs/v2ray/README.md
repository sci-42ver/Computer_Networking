# jsonv5 vs v4
- v5 still lacks something, see [1](https://github.com/v2fly/v2ray-core/issues/2448#issuecomment-1567374558) 2
## router [cfg](https://www.v2fly.org/v5/config/router.html)
- some object needs `[]` to encapsule inspired by [1](https://github.com/v2fly/v2ray-core/issues/2478#issue-1684467240), [2](https://www.v2fly.org/v5/config/dns.html#nameserverobject), [3](https://github.com/v2fly/v2ray-core/issues/2512#issue-1708855132), TODO how this is shown in the `go` code.
  Use `sudo /usr/bin/v2ray test -format=jsonv5 -c ~/v2ray_v5_ipv6_config.json` to test json file or `=json` for `v4`.
```json
// right
        "domain": [
          {
            "type": "Full",
            "value": "chat.openai.com"
          }
        ]
// this is wrong, will throw 
// Test failed: main/commands: failed to load config: [/home/czg_arch/v2ray_v5_ipv6_config.json] > infra/conf/v5cfg: unable to build config > infra/conf/v5cfg: failed to parse Router config > common/registry: unable to parse json content > json: cannot unmarshal object into Go value of type []json.RawMessage
        "domain":
          {
            "type": "Full",
            "value": "chat.openai.com"
          }

// notice geoip, domain can't be in the same context, then also geodoamin because geodomain can contain domain
        // "geoip": [
        //   {
        //     "code": "private",
        //     "inverseMatch": false,
        //     "filePath": "../../usr/share/v2ray/geoip.dat"
        //   },
        //   {
        //     "inverseMatch": false,
        //     "cidr": [
        //       {
        //         "ipAddr": "192.168.0.0",
        //         "prefix": 16
        //       }
        //     ]
        //   }
        // ],
        "domain": [
          // {
          //   "type": "Regex",
          //   "value": "\\..*speedtest.*"
          // },
          {
            "type": "Plain",
            "value": "speedtest"
          },
          {
            "type": "RootDomain",
            "value": "gnu.org"
          }
        ]
// notice for geodomain, "code" can be combined with "domain"
{
        "tag": "proxy",
        "geoDomain": [
          {
            "code": "telegram",
            "filePath": "../../usr/share/v2ray/geosite.dat"
          },
          // {
          //   "code": "geolocation-!cn",
          //   "filePath": "../../usr/share/v2ray/geosite.dat"
          // },
          {
            "domain": [
              {
                "type": "Plain",
                "value": "openai.com"
              },
              {
                "type": "Full",
                "value": "chat.openai.com"
              }
            ]
          }
        ]
      },
```
## dat
- [this](https://github.com/Loyalsoldier/geoip) and [this self-generated](https://github.com/v2fly/domain-list-community#generate-dlcdat-manually) seems to not apply to the v5
  - TODO for the dat preinstalled, `geoip:us` no use.
## TODO
- v2rayN generated config
  - [routeonly](https://github.com/XTLS/Xray-core/issues/1565)
- Static hosts usage
```json
// v4
"dns": {
    // Static hosts, similar to hosts file.
    // "hosts": {
    //   // Match v2fly.org to another domain on CloudFlare. This domain will be used when querying IPs for v2fly.org.
    //   "domain:v2fly.org": "www.vicemc.net",
    //   // The following settings help to eliminate DNS poisoning in mainland China.
    //   // It is safe to comment these out if this is not the case for you.
    //   "domain:github.io": "pages.github.com",
    //   "domain:wikipedia.org": "www.wikimedia.org",
    //   "domain:shadowsocks.org": "electronicsrealm.com"
    // },
}
```
- `Test failed: main/commands: failed to load config: [/home/czg_arch/v2ray_v5_ipv6_config.json] > infra/conf/v5cfg: unable to build config > infra/conf/v5cfg: failed to parse DNS config > common/registry: unable to parse json content > unknown field "expectIPs" in v2ray.core.app.dns.SimplifiedNameServer` for `expectIPs`
- policy in v5
```json
// Policy controls some internal behavior of how V2Ray handles connections.
  // It may be on connection level by user levels in 'levels', or global settings in 'system.'
  // "policy": {
  //   // Connection policys by user levels
  //   "levels": {
  //     "0": {
  //       "uplinkOnly": 0,
  //       "downlinkOnly": 0
  //     }
  //   },
  //   "system": {
  //     "statsInboundUplink": false,
  //     "statsInboundDownlink": false,
  //     "statsOutboundUplink": false,
  //     "statsOutboundDownlink": false
  //   }
  // },
  // // Stats enables internal stats counter.
  // // This setting can be used together with Policy and Api. 
  // //"stats":{},
  // // Api enables gRPC APIs for external programs to communicate with V2Ray instance.
  // //"api": {
  // //"tag": "api",
  // //"services": [
  // //  "HandlerService",
  // //  "LoggerService",
  // //  "StatsService"
  // //]
  // //},
  // // You may add other entries to the configuration, but they will not be recognized by V2Ray.
  // "other": {}
```
- For [this](https://ipv6-test.com/), `"domainStrategy": "AsIs"` will show "IPv4" not supported, but `"domainStrategy": "IpIfNonMatch"` will show ipv4 support.
  "https://www.test-ipv6.com/" works well for both.
```bash
$ sudo vim /var/log/v2ray/error.log
# both cfgs show
app/dispatcher: taking detour [direct] for [tcp:v6-zone4.ipv6-test.com:443]
...
app/dispatcher: taking detour [direct] for [tcp:v4.ipv6-test.com:443]
```
# miscs
- one [possible bug](https://github.com/v2fly/v2ray-core/discussions/2755) has been fixed in v5
```json
  "router": {
    "domainStrategy": "IpIfNonMatch",
    // "domainStrategy": "UseIp",
    // "domainStrategy": "AsIs",
    "rule": [
      {
        "tag": "direct",
        "domain": [
          {
            "type": "Plain",
            "value": "ipv6"
          }
        ]
      },
      ...
    ]
  }
```
  will output
```bash
$ sudo vim /var/log/v2ray/error.log
# search DNS, mostly not go through DNS and just use the domain to match the rules
2023/11/12 17:19:57 [Info] app/dns: DNS: created UDP client initialized for [2001:4860:4860::8888]:53
2023/11/12 17:19:57 [Info] app/dns: DNS: client [2001:4860:4860::8888] uses clientIP 5.6.7.8 
```
# notice
- the routing rule [order](https://github.com/Loyalsoldier/v2ray-rules-dat#geositedat-1) matters
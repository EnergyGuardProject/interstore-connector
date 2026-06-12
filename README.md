# Interstore Connector — Deployment Notes

Setup and configuration reference for the Interstore Data Space Connector deployed on `energyguard.epu.ntua.gr`.

---

## Public Endpoints

| Service | Public URL | Notes |
|---|---|---|
| Connector UI | https://interstore-connector-ui.energy-guard.eu | HTTPS via nginx proxy |
| Local API | https://interstore-connector-api.energy-guard.eu/api | Used in connector UI settings |
| ECC (inter-connector) | https://interstore-connector-ecc.energy-guard.eu/data | Used in connector UI settings |
| ECC (direct) | https://energyguard.epu.ntua.gr:8889 | Port 8889 open in UFW |

---

## Why These Services Need to Be Public

| Port | Service | Purpose |
|---|---|---|
| 8080 | connector-ui | Makes the connector dashboard accessible over HTTPS. Allows the UI to call the local API from a secure context. |
| 8889 | ecc-provider | Makes your connector reachable by remote TEF nodes for contract negotiation and data transfer. |
| 30001 | local-api | Proxies the local API to HTTPS so the UI (secure context) can call it without CORS errors. |

---

## Docker Network

Add network to service recipes in `docker-compose.yml`:

```yaml
networks:
  - nginxproxy_energyguard_net

networks:
  nginxproxy_energyguard_net:
    external: true
```

---

## Firewall (UFW)

```
     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    147.102.6.0/24            
[ 2] 22                         ALLOW IN    147.102.131.0/24          
[ 3] 22                         ALLOW IN    147.102.136.0/26          
[ 4] 7474/tcp                   ALLOW IN    Anywhere                  
[ 5] 7687/tcp                   ALLOW IN    Anywhere                  
[ 6] 5000                       DENY IN     Anywhere                  
[ 7] 443                        ALLOW IN    Anywhere                  
[ 8] 8889/tcp                   ALLOW IN    Anywhere                  
[ 9] 8082                       ALLOW IN    Anywhere                  
[10] 80/tcp                     ALLOW IN    Anywhere                  
[11] 30001/tcp                  ALLOW IN    Anywhere                  
[12] 7474/tcp (v6)              ALLOW IN    Anywhere (v6)             
[13] 7687/tcp (v6)              ALLOW IN    Anywhere (v6)             
[14] 5000 (v6)                  DENY IN     Anywhere (v6)             
[15] 443 (v6)                   ALLOW IN    Anywhere (v6)             
[16] 8889/tcp (v6)              ALLOW IN    Anywhere (v6)             
[17] 8082 (v6)                  ALLOW IN    Anywhere (v6)             
[18] 80/tcp (v6)                ALLOW IN    Anywhere (v6)             
[19] 30001/tcp (v6)             ALLOW IN    Anywhere (v6)```

---

## Nginx Proxy Manager - Proxy Hosts

**1. Connector UI - 8080**
- Click Proxy Hosts > Add Proxy Host
- Domain Names: `interstore-connector-ui.energy-guard.eu`
- Scheme: `http`
- Forward Hostname/IP: `connector-ui`
- Forward Port: `8080`
- SSL tab > Request a new SSL certificate > enable Force SSL
- Click Save

**2. Local API - 30001**
- Proxy Hosts > Add Proxy Host
- Domain Names: `interstore-connector-api.energy-guard.eu`
- Scheme: `http`
- Forward Hostname/IP: `local-api`
- Forward Port: `30001`
- SSL tab > Request new certificate > Force SSL on
- Save

**3. Execution Core Container (ECC) - 8889**
- Proxy Hosts > Add Proxy Host
- Domain Names: `interstore-connector-ecc.energy-guard.eu`
- Scheme: `https`
- Forward Hostname/IP: `ecc-provider`
- Forward Port: `8889`
- SSL tab > Request new certificate > Force SSL on
- Save

---

## Connector UI Settings

```
- Local API URL: https://interstore-connector-api.energy-guard.eu/api
- Data App URL: https://be-dataapp-provider:8083
- ECC URL: https://interstore-connector-ecc.energy-guard.eu/data
```

---

## Connectivity Check

To verify inter-connector connectivity with a remote TEF node (URL provided by ED):

Sanity check based on INSTALLATION.md:

```bash
$ curl -k -v https://interstore-connector-api.energy-guard.eu/api
* Host interstore-connector-api.energy-guard.eu:443 was resolved.
* IPv6: (none)
* IPv4: 147.102.6.166
*   Trying 147.102.6.166:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* ALPN: server accepted http/1.1
* Established connection to interstore-connector-api.energy-guard.eu (147.102.6.166 port 443) from 192.168.1.14 port 61891
* using HTTP/1.x
> GET /api HTTP/1.1
> Host: interstore-connector-api.energy-guard.eu
> User-Agent: curl/8.17.0
> Accept: */*
>
* Request completely sent off
* schannel: remote party requests renegotiation
* schannel: renegotiating SSL/TLS connection
* schannel: SSL/TLS connection renegotiated
* schannel: remote party requests renegotiation
* schannel: renegotiating SSL/TLS connection
* schannel: SSL/TLS connection renegotiated
< HTTP/1.1 302
< Server: openresty
< Date: Wed, 10 Jun 2026 10:43:19 GMT
< Transfer-Encoding: chunked
< Connection: keep-alive
< Location: http://interstore-connector-api.energy-guard.eu/api/
< Strict-Transport-Security: max-age=63072000;includeSubDomains; preload
< X-Served-By: interstore-connector-api.energy-guard.eu
<
* Connection #0 to host interstore-connector-api.energy-guard.eu:443 left intact
```

A successful response returns `302`. A `400/401/404` indicates a firewall or configuration issue.

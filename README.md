1. connector-ui (8080)
Makes the connector dashboard accessible at https://interstore-connector-ui.energy-guard.eu with a valid SSL certificate. This is the critical step that fixes the localhost:30001 CORS error — because the page becomes a secure context (HTTPS), Chrome will allow the UI to call the local API.

2. ecc-provider (8889) (PROVIDER_PORT)
Makes your connector reachable by the TEF node. This is the port the remote connector calls to negotiate contracts and transfer data.
- "${PROVIDER_PORT}:${INTERNAL_REST_PORT}"

3. ecc-provider (8090) (INTERNAL_REST_PORT)
Exposes your connector's public REST API so the TEF node can discover what data you offer.
- "${PROVIDER_PORT}:${INTERNAL_REST_PORT}"

4. local_api (30001)
Proxies the local API to HTTPS, allowing the UI (as a secure context) to call it without CORS errors.

In short: without these, your connector exists only locally. With these, it becomes a publicly reachable data space participant that the TEF node can connect to and exchange data with.

add these to local-api, connector-ui and ecc-provider docker compose recipes:
```
    networks:
      - nginxproxy_energyguard_net

networks:
  nginxproxy_energyguard_net:
    external: true
```

``` bash
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
[ 9] 8090/tcp                   ALLOW IN    Anywhere                  
[10] 8082                       ALLOW IN    Anywhere                  
[11] 80/tcp                     ALLOW IN    Anywhere                  
[12] 30001/tcp                  ALLOW IN    Anywhere                  
[13] 7474/tcp (v6)              ALLOW IN    Anywhere (v6)             
[14] 7687/tcp (v6)              ALLOW IN    Anywhere (v6)             
[15] 5000 (v6)                  DENY IN     Anywhere (v6)             
[16] 443 (v6)                   ALLOW IN    Anywhere (v6)             
[17] 8889/tcp (v6)              ALLOW IN    Anywhere (v6)             
[18] 8090/tcp (v6)              ALLOW IN    Anywhere (v6)             
[19] 8082 (v6)                  ALLOW IN    Anywhere (v6)             
[20] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
[21] 30001/tcp (v6)             ALLOW IN    Anywhere (v6)
```

1. UI — 443 → 8080 (Proxy Host)
- Click Proxy Hosts → Add Proxy Host
- Domain Names: interstore-connector-ui.energy-guard.eu
- Scheme: http
- Forward Hostname/IP: connector-ui
- Forward Port: 8080
- Go to SSL tab → Request a new SSL certificate → enable Force SSL
- Click Save
2. ECC inter-connector — 8889 → 8889 (Stream)
- Click Streams → Add Stream
- Incoming Port: 8889
- Forward Host: ecc-provider
- Forward Port: 8889
- TCP selected, UDP off
- Click Save
3. ECC public API — 8090 → 8449 (Stream)
- Click Streams → Add Stream
- Incoming Port: 8090
- Forward Host: ecc-provider
- Forward Port: 8449
- TCP selected, UDP off
- Click Save
4. Proxy Host for Local API
- Proxy Hosts → Add Proxy Host
- Domain Names: interstore-connector-api.energy-guard.eu
- Scheme: http
- Forward Hostname/IP: local-api
- Forward Port: 30001
- SSL tab → Request new certificate → Force SSL on
- Save
5. Proxy Host for Local Execution Core Container (ECC)
- Proxy Hosts → Add Proxy Host
- Domain Names: interstore-connector-ecc.energy-guard.eu
- Scheme: http
- Forward Hostname/IP: ecc-provider
- Forward Port: 8889
- SSL tab → Request new certificate → Force SSL on
Advanced tab (gear icon) → add this to disable upstream cert verification: proxy_ssl_verify off;
- Save
6. Proxy Host for dataapp provider
- Proxy Hosts → Add Proxy Host
- Domain Names: interstore-connector-ecc.energy-guard.eu
- Scheme: https
- Forward Hostname/IP: be-dataapp-provider
- Forward Port: 8083
- SSL tab → Request new certificate → Force SSL on
Advanced tab (gear icon) → add this to disable upstream cert verification: proxy_ssl_verify off;


At connector UI settings:

```
Local Api Url: https://interstore-connector-api.energy-guard.eu/api
Data app:     https://be-dataapp-provider:8083 (https://interstore-connector-dataapp-provider.energy-guard.eu)
Ecc Url:      https://energyguard.epu.ntua.gr:8889/data (https://interstore-connector-ecc.energy-guard.eu/data)
```

Sanity check based on INSTALLATION.md:
```
(.venv) atzortzis@energyguard:~/Data-Space-Connector$ curl -k -v --max-time 10 https://energyguard.epu.ntua.gr:8889/data
* Host energyguard.epu.ntua.gr:8889 was resolved.
* IPv6: (none)
* IPv4: 147.102.6.166
*   Trying 147.102.6.166:8889...
* Connected to energyguard.epu.ntua.gr (147.102.6.166) port 8889
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519 / RSASSA-PSS
* ALPN: server did not agree on a protocol. Uses default.
* Server certificate:
*  subject: C=IT; ST=Italy; L=Lecce; O=Engineering Ingegneria Informatica SpA; OU=R&D; CN=execution-core-container
*  start date: Jul  8 10:20:57 2022 GMT
*  expire date: Jul  8 10:20:57 2026 GMT
*  issuer: C=IT; ST=Italy; L=Lecce; O=Engineering Ingegneria Informatica SpA; OU=R&D; CN=execution-core-container
*  SSL certificate verify result: self-signed certificate (18), continuing anyway.
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
* using HTTP/1.x
> GET /data HTTP/1.1
> Host: energyguard.epu.ntua.gr:8889
> User-Agent: curl/8.5.0
> Accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< HTTP/1.1 401 Unauthorized
< Content-Type: text/plain
< Accept: */*
< User-Agent: curl/8.5.0
< Transfer-Encoding: chunked
< Server: Jetty(9.4.33.v20201020)
< 
* Connection #0 to host energyguard.epu.ntua.gr left intact Access denied
```

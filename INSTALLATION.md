# Data space connector architecture

The Data Space architecture is based on a federated connector-based approach, enabling secure and sovereign data exchange among participating entities. The core component of this architecture is the Interstore Data Space Connector ([GitHub - Horizont-Europe-Interstore/Data-Space-Connector: Energy Data Space Connector Docker Project · GitHub](https://github.com/Horizont-Europe-Interstore/Data-Space-Connector)), which is deployed by each participating Testing and Experimentation Facility (TEF).

Each TEF installs and operates its own instance of the Interstore Data Space Connector. This connector serves as the primary interface through which the participant interacts with the data space ecosystem. Through the connector, participants can publish, request, and consume data services while maintaining control over their local data assets.

The Interstore connector supports several key functions within the ecosystem, including:

- Service subscription management, enabling participants to create and manage requests for data services.
- Data provisioning, allowing participants to expose datasets or services to other members of the data space.
- Data consumption and provision, enabling authorized access to services/folders provided by other connectors within the ecosystem.
- Secure data exchange, ensuring that interactions between participants follow agreed policies and access rules.

While each connector operates independently at the participant level, interoperability and coordination across the ecosystem are achieved through the OneNet Middleware ([GitHub - european-dynamics-rnd/Onenet_Middleware · GitHub](https://github.com/european-dynamics-rnd/Onenet_Middleware)). The OneNet middleware acts as a shared platform layer to which all connectors connect.

The middleware provides a set of common services required for the operation of the data space, including:

- **User Management** – managing identities and authentication of users interacting with the system.
- **Access Management** – defining and enforcing authorization policies for data access and service usage.
- **Federated Catalogue** – enabling the discovery of available datasets and services across all participating connectors.
- **Shared Vocabularies** – supporting semantic interoperability through common data models and controlled vocabularies.

This architecture enables a federated and decentralized data-sharing environment in which each participant retains sovereignty over its data while benefiting from shared governance, interoperability services, and discoverability mechanisms provided by the middleware layer.

By combining locally controlled connectors with centralized federation services, the system supports scalable and secure collaboration between multiple TEFs participating in the data space ecosystem.

---

# Dataspace connector installation

1. To install the Interstore DS connector first check the following VM requirements

2. Download DS connector.
```bash
git clone https://github.com/Horizont-Europe-Interstore/Data-Space-Connector
```

3. In the file `Data-Space-Connector/docker/.env` change last line (`API_URL`) to point to Onenet middleware installed on ED premises.

```
API_URL=https://enertef-dataspace-middleware-api.eurodyn.com/api
```
4. Update the `connector-ui` service in the `docker_compose.yml` with the following to include the EnerTEF style connector UI pallet theme. Change the ports if necessary (if you don't want the connector UI to listen on external port 8080).

```yaml
connector-ui:
    image: ktouloumis/connector-ui-branded:latest
    container_name: connector-ui
    volumes:
-	./connector-ui/nginx-conf/ui-nginx.conf:/etc/nginx/templates/ui-nginx.conf.template
    ports:
      - "8080:8080"
    restart: always
    networks:
      - provider
      - consumer
    depends_on:
      - be-dataapp-provider
      - be-dataapp-consumer
```

5. Build the connector.
```bash
docker compose up -d –build
```

6. Contact ED to create you a user account for the DS connector in the Middleware.

7. **(OPTIONAL)** The dataspace connector comes with 2 services: i) a provider and ii) a consumer service. So now you should be able to test your connector locally with 2 different users by configuring one user as provider and one user as consumer. A data transfer should work between them:
   1. For provider user put the following settings in the connector UI.
    ```
    - Local Api URL: https://your_local_api_public_url/api
    - Data app URL: https://be-dataapp-provider:8083
    - ECC url: https://ecc-provider:8889/data
    ```
   2. For consumer user put the following settings in the connector UI.
    ```
    - Local API url: https://your_local_api_public_url/api
    - Data app URL: https://be-dataapp-consumer:8084
    - ECC url: https://ecc-consumer:8889/data
    ```

8. To allow the connector to communicate with a data space connector on another VM, your connector settings should be configured to make the connector work as a provider and also they should be forwarded to https to make them accessible through the web. Please make the following reverse proxy configuration.

```
- Local ECC (port 8889):https://your_local_domain:8889-> https://your_ecc_public_url  
- Local API (port 30001):http://your_local_domain:30001/api-> https://your_localapi_public_url/api 
- Local GUI Port: http:// your_local_domain:8080 -> https://your_connector_ui_public_ip
```

9. Then in the connector settings put.
```
•	Local API: https:// your_localapi_public_url/api
•	Data app: https://be-dataapp-provider:8083 
•	ECC URL: https://your_ecc_public_url/data 
```

10. Reach a TEF node already having the connector to make a data transfer between your connector and theirs.

```
Troubleshooting
To check if the connection with the other VM works use:
$curl -k -v --max-time 10 https://remote_api_url 
if the output contains code 400/401/404 the connection is not established so check firewall configuration. 
An output like the following indicates success
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
< HTTP/2 302
< server: openresty
< date: Fri, 06 Mar 2026 12:00:00 GMT
< location: http://enertef-dataspace-localapi.enertef.epu.ntua.gr/api/
< strict-transport-security: max-age=63072000;includeSubDomains; preload
< x-served-by: enertef-dataspace-localapi.enertef.epu.ntua.gr
* Connection #0 to host enertef-dataspace-localapi.enertef.epu.ntua.gr left intact
```

---

# Data transfer example

## Provide Data

1. To provide data firstly, a new offered service should be created. Under **Services > My offered services** by click the button **"New Data Service+"**. Then define the details of the offering. The field business object is necessary, select one from the listed ones.

   *Figure 1 Create service offering*

   *Figure 2 Service offerings details*

2. To provide data to a service offering under **Data Exchange > Provide Data** click **"New Data+"** and then select the file you want to provide.

   *Figure 3 New Data provide*

   *Figure 4 Upload Data*

3. To accepted incoming requests from consumers under **Services > Requests** click the pencil button and change the status to accept. Then click save.

   *Figure 5 My Requests*

   *Figure 6 Accept Request*

## Consume Data

1. In order to consume data from a particular service offering first you have to subscribe to the offering and the provider must accept your request. In the menu under **Services > My subscriptions** where all the offerings you have subscribed are listed, click **"New data Subscription+"**. Then click **"Select offering"** select one from the listed offerings.

   *Figure 7 My Subscriptions*

   *Figure 8 Subscribe to offering*

2. Once the provider has accepted the subscription request in the menu under **Data exchange > Consume Data** select the offering you would like to download data from and click download data.

   *Figure 9 Consume data listings*

   *Figure 10 Consume Data*
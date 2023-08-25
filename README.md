# Sample Deployment for Alfresco Community with Alfresco Identity Service and Keycloak

**Alfresco Identity Service**, based in Open Source Identity and Access Management [Keycloak](https://www.keycloak.org/), is available for both Community and Enterprise ACS releases.

When using **Alfresco Identity Service**, different [Alfresco Authentication Mechanims](https://docs.alfresco.com/content-services/community/admin/auth-sync/) can be configured.

This project provides a *sample* Docker Compose template for Alfresco Community 7.4 with Alfresco Identity Service 2.0.0 and Keycloak 22.0. Note that deploying the product in *production* environments would require additional configuration.

Since EOL for Alfresco Identity Service has been [announced](https://hub.alfresco.com/t5/alfresco-content-services-blog/alfresco-identity-service-end-of-life/ba-p/317195), this project can be used to understand the transition from Alfresco Identity Service to Keycloak. Alfresco Identity Service 2.0.0 will be the latest release of this product. The recommendation is to switch to Keycloak eventually.


## Docker Compose

This project includes sample configuration to use Alfresco Identity Service 2.0.0 or Keyclock 22.0 with Alfresco 7.4.1. Both configurations are equivalent, so you can move from AIS 2.0 to Keycloak with minimal impact.

**alfresco-community-aims**

Alfresco Repository is configured to use Alfresco Identity Service 2.0.0 using the realm `alfresco`

```
authentication.chain=identity-service1:identity-service
identity-service.enable-basic-auth=true
identity-service.auth-server-url=http://${HOST_IP}:8999
identity-service.realm=alfresco
identity-service.resource=alfresco
```

Alfresco Identity Service 2.0.0 is configured with default credentials `admin`/`admin` importing a default Alfresco Realm and using plain HTTP.

```
alfresco-identity-service:
    image: alfresco/alfresco-identity-service:2.0.0
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
    command: "
        start --import-realm --hostname=$HOST_IP --hostname-port=8999 --http-enabled=true --hostname-strict-https=false
      "
    volumes:
      - ./config/alfresco-realm.json:/opt/keycloak/data/import/alfresco-realm.json
```

**alfresco-community-keycloak**

The only difference with the previous `compose.yaml` is that the Docker Image for the Identity Service is direclty the one provided by Keycloak.

```
alfresco-identity-service:
    image: quay.io/keycloak/keycloak:22.0
```

So, in this case, moving from Alfresco Identity Service to Keycloak is as simple as using a different Docker Image.

>> Depending on your configuration, additional steps may be required.

Both Docker Compose templates ([alfresco-community-aims](alfresco-community-aims) and [alfresco-community-keycloak](alfresco-community-keycloak)) include following files:

```
.
├── .env
├── config
│   ├── alfresco-realm.json
│   └── nginx.conf
└── compose.yaml
```

* `docker-compose.yml` is a regular ACS Docker Compose, including Alfresco Identity Service or Keycloak for Authentication
* `config/alfresco-realm.json` includes a sample configuration for Alfresco Identity Service or Keycloak, despite you can create your own configuration using the Keycloak Admin Web Page
* `config/nginx.conf` includes configuration for the NGINX Web Proxy
* `.env` includes Docker Image tag names and a HOST_IP variable (remember to add your local computer IP to this property before running the project)

## Using

>> Note: Add your local computer IP to `.env` variable HOST_IP before running Docker Compose.

```
$ docker-compose up --build --force-recreate
```

## Service URLs

http://localhost:8080/

ACA

* user: admin
* password: admin

http://localhost:8080/share

Share

* user: admin
* password: admin

http://localhost:8080/alfresco

Alfresco Repository

* user: admin
* password: admin

http://${HOST_IP}:8999

Keycloak

* user: admin
* password: admin

**Default configuration for Alfresco Identity Service or Keycloak**

Users

* admin / admin
* test / admin

Roles
* test_role
* default-roles-alfresco

Groups

* admin
* testgroup

## Additional resources

Additional information on Alfresco Identity Service configuration is available in https://docs.alfresco.com/identity-service/latest/

For instance, following properties can be used from `alfresco-global.properties`

```
identity-service.authentication.enabled=
identity-service.enable-basic-auth=
identity-service.authentication.defaultAdministratorUserNames=
identity-service.authentication.validation.failure.silent=
identity-service.auth-server-url=
identity-service.realm=
identity-service.resource=
identity-service.public-client=
identity-service.ssl-required=
identity-service.enable-pkce=
identity-service.credentials.secret=
identity-service.credentials.provider=
```

Tutorials on how to configure different Authetication Subsystems are available in:

* https://docs.alfresco.com/identity-service/latest/tutorial/sso/saml/
* https://docs.alfresco.com/identity-service/latest/tutorial/sso/ldap/
* https://docs.alfresco.com/identity-service/latest/tutorial/sso/kerberos/

## Related projects

If you want to discover some other features that can be used with Keycloak, check this Community project:

https://github.com/Acosix/alfresco-keycloak

Additional details are provided by [@AFaust](https://github.com/afaust) in TTL #133: https://www.alfresco.com/events/webinars/tech-talk-live-133

# IAM components

## Overview and subcomponents

The IAM framework enables actors in the DOME ecosystem to authenticate
into DOME services (i.e., the DOME Portal, federated marketplace portals or TM Forum APIs
implemented on top of the DOME Persistence Layer) and help to manage proper access to those
services based on their profiles. This IAM framework relies on Verifiable
Credentials/Verifiable Presentations and leverages the Trust framework to provide an
efficient, scalable, and decentralised IAM that participants can use. The Trust and IAM
framework implemented in DOME is not only for managing authentication and authorization in
the interaction with the DOME services, but it is also available for data/app services
which may use it for managing the interactions between them and their users.

IAM framework comprises following components:
* [Trusted-Issuers-List (TIL)](https://github.com/fiware/trusted-issuers-registry). This
service provides an [EBSI Trusted Issuers Registry](https://hub.ebsi.eu/apis/pilot/trusted-issuers-registry/v4) implementation to act as the
Trusted-List-Service in the DSBA Trust and IAM Framework. In addition, a Trusted-Issuers-List
API to manage the issuers is provided.
* [VCVerifier](https://github.com/fiware/vcverifier). VCVerifier provides the necessary
endpoints to offer SIOP-2/OIDC4VP compliant authentication flows. It exchanges
VerifiableCredentials for JWT, that can be used for authorization and authentication in
down-stream components.
* [Credentials Config Service (CCS)](https://github.com/fiware/credentials-config-service).
It manages and provides information about services and the credentials they are using. It
returns the scope to be requested from the wallet per service and the credentials and
issuers that are considered to be trusted for a certain service.
* [Keycloak-VC-Issuer](https://github.com/fiware/keycloak-vc-issuer). The Keycloak-VC-Issuer
is plugin for Keycloak to support SIOP-2/OIDC4VP clients and issue VerifiableCredentials
through the OIDC4VCI-Protocol to compliant wallets.
* [PDP](https://github.com/fiware/dsba-pdp). Implementation of a Policy-Decision Point,
evaluating Json-Web-Tokens containing VerifiableCredentials in an DSBA-compliant way. It
also supports the evaluation in the context of i4Trust.
* [Keyrock](https://github.com/ging/fiware-idm). Keyrock is the FIWARE component responsible
for Identity Management. Using Keyrock (in conjunction with other security components)
enables you to add OAuth2-based authentication and authorization security to your services
and applications.
* [Kong Plugins](https://github.com/fiware/kong-plugins-fiware). These allow to extend the
API Gateway Kong by further functionalities required for FIWARE-based environments. Kong
Gateway is a lightweight, fast, and flexible cloud-native API gateway. An API gateway is a
reverse proxy that lets you manage, configure, and route requests to your APIs.

## Infrastructure requirements

The recommended order of deploying components is shown in following sections, since some
of the components are dependent on the others.

* All the database components should obviously be installed first.
* Waltid component is needed by Keycloak-VC-Issuer and VCVerifier, so the service URL
of it should be correctly mapped in corresponding dependent components.
* VCVerifier requires Waltid, TIL, CCS to be already deployed and available, so their
service URLs should be added to the verifier's deployment.
* PDP relies on Keyrock component as an authorization registry, so access to it should
be added as well.
* Kong, as an API Gateway, needs URLs of PDP (to get a decision whether a protected
resource can be accessed or not) and all services, which should be protected by it.

## How to deploy

All these components will be deployed through [Helm Charts](https://helm.sh/). Before deploying
components, make sure to adapt config values according to your needs (names, URLs, etc.).
To install a component, run following commands (assuming you've opened _applications_
folder in the terminal):
```shell
helm dependency build <component_folder>
helm install <any_name> <component_folder>
```
For example, for Waltid component these commands could look like this:
```shell
helm dependency build walt-id/
helm install waltid walt-id/
```

### Deploying VC Issuer

* **Waltid** is the component that is used for creating Verifiable Credentials that match a
  given template using a defined key. The template is provided on startup and the key is
  registered by the Keycloak VC Issuer Plugin. An example can be found
  [here](./applications/walt-id).
* Persistence for Keycloak-VC-Issuer can be external or internal. In case of external,
  the **Postgres** component should be deployed and database access settings need to be added
  to the VC Issuer deployment. An example of deploying postgres instance can be found
  [here](./applications/postgres).
* Since **Keycloak-VC-Issuer** is a plugin for Keycloak, an instance of Keycloak should
  be deployed, which can be configured to install the plugin. An example can be found
  [here](./applications/keycloak).

### Deploying TIL, CCS and VCVerifier

* **MySQL**. Trusted-Issuers-List requires an SQL database. It currently supports
  MySql-compatible DBs and H2 (as an In-Memory DB for dev/test purposes). Migrations are
  applied via [flyway](https://flywaydb.org/), see the
  [migration-scripts](https://github.com/FIWARE/trusted-issuers-list/tree/main/src/main/resources/db/migration)
  for the schema. To save resources, Trusted-Issuers-List and Credentials-Config-Service can
  use the same MySQL deployment (but each of them needs its own database). Example can be found
  [here](./applications/mysql).
* **Trusted-Issuers-List**. The latest example can be found
  [here](./applications/trusted-issuers-list).
* **Credentials-Config-Service**. An example can be found
  [here](./applications/credentials-config-service).
* **VCVerifier**. Example of deploying verifier can be found [here](./applications/verifier).

### Deploying Keyrock

Keyrock requires and SQL database. You can use the same MySQL deployment, which was created
for TIL and CCS components. Example of deploying a Keyrock instance can be found
[here](./applications/keyrock).

### Deploying DSBA-PDP

Example can be found [here](./applications/dsba-pdp).

### Deploying Kong plugins

There is a specific Kong build, that incorporates all these plugins. When using this build,
the plugins do not need to be installed separately. It can be found on
[Quay](https://quay.io/fiware/kong). Example of deploying is shown [here](./applications/kong).

## How to configure

These provided examples have configurations for demonstration purpose, but you can configure
these components according to your needs. The configuration information can be found either
on the components chart repository or on the component repository itself. Most of the charts
can be found either [here](https://github.com/FIWARE/helm-charts) or
[here](https://github.com/i4Trust/helm-charts). All links to the corresponding projects
you can find in the [overview](#overview-and-subcomponents) section.
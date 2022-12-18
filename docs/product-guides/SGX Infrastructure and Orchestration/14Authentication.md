# Authentication

Authentication is centrally managed by the Authentication and Authorization Service (AAS). This service uses a Bearer Token authentication method. This service also centralizes the creation of roles and users, allowing much easier management of users, passwords, and permissions across all Intel® SecL-DC services.

To make an API request to an Intel® SecL-DC service, an authentication token is required. API requests must now include an Authorization header with a valid token

The token is issued by AAS and expires after a set amount of time. This token may be used with any Intel® SecL-DC service and will carry the appropriate permissions for the role(s) assigned to the account the token was generated for.

The SKC solution involves AAS deployments for 2 different domains: the CSP domain and the tenant domain. There is no trust relationship between the 2 deployments.

In SKC, the accounts of the SGX Services are created at install time. However, CSP admin users must obtain AAS tokens to invoke admin APIs in the SGX Host Verification Service (SHVS), the SGX Hub, the SGX Caching Service (SCS) and AAS.

Similarly, the tenant admin needs AAS tokens to invoke Create, Read, Update and Delete (CRUD) APIs in KBS and admin APIs in AAS.

The following sections present how to use AAS APIs to create tokens and manage users.

## Create Token

To request a new token from the AAS:

POST https://<AAS IP or hostname>:8444/aas/v1/token

{

"username" : "<username>",

"password" : "<password>"

}

The response will be a token that can be used in the Authorization header for other requests. The length of time for which the token will be valid is configured on the AAS using the key ~AAS_JWT_TOKEN_DURATION_MINS~ (in the installation answer file during installation) or aas.jwt.token.duration.mins (configured on the AAS after installation). In both cases the value is the length of time in minutes that issued tokens will remain valid before expiring.

## User Management

Users in Intel® SecL-DC are centrally managed by the Authentication and Authorization Service (AAS). Any user may be assigned roles for any service, allowing user accounts to be fully defined by the tasks needed

### Username and Password Requirement

Passwords have the following constraints:

-   cannot be empty - ie must at least have one character

-   maximum length of 255 characters

Usernames have the following requirements:

-   Format: username[@host_name[domain]]

-   [@host_name[domain]] is optional

-   username shall be minimum of 2 and maximum of 255 characters

-   username allowed characters are alphanumeric, ., -, \_ - but cannot start with -.

-   Domain name must meet requirements of a host name or fully qualified internet host name

-   (Update it relevant to SKC)

### Create User

POST https://<IP or hostname of AAS>:8444/aas/v1/users

Authorization: Bearer <token>

{

"username" : "<username>",

"password" : "<password>"

}

### Search Users by Username

GET https://<IP or hostname of AAS>:8444/aas/v1/users?name=<value>

Authorization: Bearer <token>

### Change User Password

PATCH https://<IP or hostname of AAS>:8444/aas/v1/users/changepassword

{

"username": "<username>",

"old_password": "<old_password>",

"new_password": "<new_password>",

"password_confirm]": "<new_password>" }

### Delete User

DELETE https://<IP or hostname of AAS>:8444/aas/v1/users/<User ID>

Authorization: Bearer <token>

## Roles and Permission

Permissions in Intel® SecL-DC are managed by Roles. Roles are a set of predefined permissions applicable to a specific service. Any number of Roles may be applied to a User. While new Roles can be created, each Intel® SecL service defines permissions that are applicable to specific predetermined Roles. This means that only pre-defined Roles will actually have any permissions. Role creation is intended to allow Intel® SecL-DC services to define their permissions while allowing role and user management to be centrally managed on the AAS. When a new service is installed, it will use the Role creation functions to define roles applicable for that service in the AAS.

### Create Roles

POST https://<AAS IP or Hostname>:8444/aas/v1/roles

Authorization: Bearer <token>

{

"service": "<Service name>",

"name": "<Role Name>".

"permissions": [<array of permissions>]

}

-   Service field contains a minimum of 1 and maximum of 20 characters. Allowed characters are alphanumeric plus the special charecters -, \_, @, ., ,

-   Name field contains a minimum of 1 and maximum of 40 characters. Allowed characters are alphanumeric plus the special characters -, \_, @, ., ,

-   Service and Name fields are mandatory

-   Context field is optional and can contain up to 512 characters. Allowed characters are alphanumeric plus -, \_, @, ., ,,=,;,:,\*

-   Permissions field is optional and allow up to a maximum of 512 characters.

The Permissions array must a comma-separated list of permissions formatted as resource:action:

Permissions required to execute specific API requests are listed with the API resource and method definitions in the API documentation.

### Search Roles

GET https://<AAS IP or Hostname>:8444/aas/v1/roles?<parameter>=<value>

Authorization: Bearer <token>

Search parameters supported:

Service=<name of service>

Name=<role name>

Context=<context>

contextContains=<partial "context" string>

allContexts=<true or false> filter=false

### Delete Role

DELETE https://<AAS IP or Hostname>:8444/aas/v1/roles/<role ID> Authorization:

Bearer <token>

### Assign Role to User

POST https://<AAS IP or Hostname>:8444/aas/v1/users/<user ID>/roles

Authorization: Bearer <token>

{

"role_ids": ["<comma-separated list of role IDs>"]

}

### List Roles Assigned to User

GET https://<AAS IP or Hostname>:8444/aas/v1/users/<user ID>/roles

Authorization: Bearer <token>

### Remove Role from User

DELETE https://<AAS IP or Hostname>:8444/aas/v1/users/<user ID>/roles/<role ID>

Authorization: Bearer <token>

### Global Admin level permission

Provided Global Admin permissions to below services,

 * SGX Caching Service
 * SGX Host Verification Service
 * SGX Quote Verification Service
 * Key Broker Service

### Role Definitions

Following are the set of roles which are required during installation and runtime.

| Role Name                                                    | Permissions                                                  | Utility                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| < SHVS:HostDataUpdater: >                                    |                                                              | Used by the SGX_Agent to push host data to the SHVS          |
| < SHVS:HostsListReader: >                                    |                                                              | Used by the IHUB to retrieve the list of hosts from SHVS     |
| < SHVS:HostDataReader: >                                     |                                                              | Used by the IHUB to retrieve platform-data from SHVS         |
| < CMS:CertApprover:CN=SHVS TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                                   | Used by the SHVS to retrieve TLS Certificate from CMS        |
| < CMS:CertApprover:CN=Integration HUB TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                        | Used by the IHUB to retrieve TLS Certificate from CMS        |
| < SCS:HostDataUpdater: >                                     |                                                              | Used by the SGX_Agent to push the platform-info to SCS       |
| < SCS:HostDataReader: >                                      |                                                              | Used by the SGX_Agent to retrieve the TCB status info from SCS|
| < SCS:CacheManager: >                                        |                                                              | Used by the SCS admin to refresh the platform info           |
| < CMS:CertApprover:CN=SCS TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                                    | Used by the SCS to retrieve TLS Certificate from CMS         |
| < KBS:KeyTransfer:permissions=nginx,USA >                    |                                                              | Used by the SKC Library user for Key Transfer                |
| < CMS:CertApprover:CN=skcuser;CERTTYPE=TLS-Client>           |                                                              | Used by the SKC Library user to retrieve TLS-Client Certificate from CMS |
| < CMS:CertApprover:CN=KBS TLS Certificate;SAN=<san list>;CERTTYPE=TLS> |                                                    | Used by the KBS to retrieve TLS Certificate from CMS         |
| AAS: Administrator                                           | *:*:*                                                        | Administrator role for the AAS only. Has all permissions for AAS resources, including the ability to create or delete users and roles |
| AAS: RoleManager                                             | AAS: [roles:create:*, roles:retrieve:*, roles:search:*, roles:delete:*] | AAS role that allows all actions for Roles but cannot create or delete Users or assign Roles to Users. |
| AAS: UserManager                                             | AAS: [users:create:*, users:retrieve:*, users:store:*, users:search:*, users:delete:*] | AAS role with all permissions for Users but has no ability to create Roles or assign Roles to Users. |
| AAS: UserRoleManager                                         | AAS: [user_roles:create:*, user_roles:retrieve:*, user_roles:search:*, user_roles:delete:*] | AAS role with permissions to assign Roles to Users but cannot create delete or modify Users or Roles. |
| < SHVS:HostListManager:>                                     |                                                              | Used by the SHVS admin to delete the hosts.                  |
| < SQVS:QuoteVerifier: >                                      |                                                              | Used by the KBS service user for quote verification          |


## SGX Agent

The SGX Agent communicates with SGX Caching Service (SCS) and SGX Host Verification Service (SHVS) directly. Authentication has been centralized with the new Authentication and Authorization Service.

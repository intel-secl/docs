# Authentication

Beginning in the Intel® SecL-DC 1.6 release, authentication is centrally
managed by the Authentication and Authorization Service (AAS). This
service uses a Bearer Token authentication method, which replaces the
previous HTTP BASIC authentication. This service also centralizes the
creation of roles and users, allowing much easier management of users,
passwords, and permissions across all Intel® SecL-DC services.

To make an API request to an Intel® SecL-DC service, an authentication
token is now required. API requests must now include an Authorization
header with an appropriate token:

`Authorization: Bearer $TOKEN`

The token is issued by the AAS and will expire after a set amount of
time. This token may be used with any Intel® SecL-DC service, and will
carry the appropriate permissions for the role(s) assigned to the
account the token was generated for.



## Create Token
------------

To request a new token from the AAS:

```
POST https://<AAS IP or hostname>:30444/aas/v1/token

{
    "username" : "<username>",
    "password" : "<password>"
}
```

The response will be a token that can be used in the Authorization
header for other requests. The length of time for which the token will
be valid is configured on the AAS using the key
`AAS\_JWT\_TOKEN\_DURATION\_MINS` (in the installation answer file during
installation) or `aas.jwt.token.duration.mins` (configured on the AAS
after installation). In both cases the value is the length of time in
minutes that issued tokens will remain valid before expiring.



## User Management
---------------

Users in Intel® SecL-DC are no longer restrained to a specific service,
as they are now centrally managed by the Authentication and
Authorization Service. Any user may now be assigned roles for any
service, allowing user accounts to be fully defined by the tasks needed.

### Username and Password requirements

Passwords have the following constraints:

-   cannot be empty - i.e must at least have one character

-   maximum length of 255 characters

Usernames have the following requirements:

-   Format: username\[@host\_name\[domain\]\]

-   \[@host\_name\[domain\]\] is optional

-   username shall be minimum of 2 and maximum of 255 characters

-   username allowed characters are alphanumeric, ., -, \_ - but cannot
    start with -.

-   Domain name must meet requirements of a host name or fully qualified
    internet host name

-   Examples

    -   admin, admin\_wls, admin@wls, <admin@wls.intel.com>,
        <wls-admin@intel.com>

### Create User

```
POST https://<IP or hostname of AAS>:30444/aas/v1/users
Authorization: Bearer <token>

{
	"username" : "<username>",
	"password" : "<password>"
}
```

### Search Users by Username

```json
GET https://<IP or hostname of AAS>:30444/aas/v1/users?name=<value>
```

### Change User Password

```json
PATCH https://<IP or hostname of AAS>:30444/aas/v1/users/changepassword
Authorization: Bearer <token>
{
	"username": "<username>",
	"old_password": "<old_password>",
	"new_password": "<new_password>",
	"password_confirm": "<new_password>"
}
```

### Delete User

```json
DELETE https://<IP or hostname of AAS>:30444/aas/v1/users/<User ID>
Authorization: Bearer <token>
```



## Roles and Permissions
---------------------

Permissions in Intel® SecL-DC are managed by Roles. Roles are a set of
predefined permissions applicable to a specific service. Any number of
Roles may be applied to a User. While new Roles can be created, each
Intel® SecL service defines permissions that are applicable to specific
predetermined Roles. This means that only pre-defined Roles will
actually have any permissions. Role creation is intended to allow Intel®
SecL-DC services to define their permissions while allowing role and
user management to be centrally managed on the AAS. When a new service
is installed, it will use the Role creation functions to define roles
applicable for that service in the AAS.

### Create Role

```
POST https://<AAS IP or Hostname>:30444/aas/v1/roles
Authorization: Bearer <token>

{
    "service": "<Service name>",
    "name": "<Role Name>",
    "permissions": [<array of permissions>]
}
```

-   `service` field contains a minimum of 1 and maximum of 20 characters.
    Allowed characters are alphanumeric plus the special charecters -,
    \_, @, ., ,

-   `name` field contains a minimum of 1 and maximum of 40 characters.
    Allowed characters are alphanumeric plus the special characters -,
    \_, @, ., ,

-   `service` and `name` fields are mandatory

-   `context` field is optional and can contain up to 512 characters.
    Allowed characters are alphanumeric plus -, \_, @, ., ,,=,;,:,\*

-   `permissions` field is optional and allow up to a maximum of 512
    characters.

The Permissions array must a comma-separated list of permissions
formatted as `resource:action:`

Permissions required to execute specific API requests are listed with
the API resource and method definitions in the API documentation.

### Search Roles

```
GET https://<AAS IP or Hostname>:30444/aas/v1/roles?<parameter>=<value>
Authorization: Bearer <token>
```

Search parameters supported:

```
Service=<name of service>
Name=<role name>
Context=<context>
contextContains=<partial "context" string>
allContexts=<true or false>
filter=false
```

### Delete Role

```
DELETE https://<AAS IP or Hostname>:30444/aas/v1/roles/<role ID>
Authorization: Bearer <token>
```

### Assign Role to User

```
POST https://<AAS IP or Hostname>:30444/aas/v1/users/<user ID>/roles
Authorization: Bearer <token>

{
	"role_ids": ["<comma-separated list of role IDs>"]
}
```

### List Roles Assigned to User

```
GET https://<AAS IP or Hostname\>:30444/aas/v1/users/<user ID>/roles
Authorization: Bearer <token>
```

### Remove Role from User

```
DELETE https://<AAS IP or Hostname>:30444/aas/v1/users/<userID>/roles/<role ID>
Authorization: Bearer <token>
```

### Role Definitions

The following roles are created during installation (or by the
CreateUsers script) and exist by default.

| **Role Name**             | **Permissions**                                              | **Utility**                                                  |
| ------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| TA:Administrator          | TA:\*:\*                                                     | Used by the Verification Service to access Trust Agent APIs, including retrieval of TPM quotes, provisioning Asset Tags and IMA Flavor, etc. |
| HVS:ReportSearcher        | HVS: \[reports:search:\*"]                                   | Used by the Integration Hub to retrieve attestation reports from the Verification Service |
| KBS:Keymanager            | KBS: \["keys:create:\*", "keys:transfer:\*"\]                | Used by the WPM to create and retrieve symmetric encryption keys to encrypt workload images |
| WLS:KeysCreator           | WLS: \["keys:create:\*"                                      | Used by the Workload Agent during Container Confidentiality flows to retrieve the key. |
| HVS: ReportCreator        | HVS: \["reports:create:\*"\]                                 | Used by the Workload Service to create new attestation reports on the Verification Service as part of Workload Confidentiality key retrievals. |
| Administrator             | \*:\*:\*                                                     | Global administrator role used for the initial administrator account. This role has all permissions across all services, including permissions to create new roles and users. |
| AAS: Administrator        | \*:\*:\*                                                     | Administrator role for the AAS only. Has all permissions for AAS resources, including the ability to create or delete users and roles. |
| AAS: RoleManager          | AAS: \[roles:create:\*, roles:retrieve:\*, roles:search:\*, roles:delete:\*\] | AAS role that allows all actions for Roles, but cannot create or delete Users or assign Roles to Users. |
| AAS: UserManager          | AAS: \[users:create:\*, users:retrieve:\*, users:store:\*, users:search:\*, users:delete:\*\] | AAS role with all permissions for Users, but has no ability to create Roles or assign Roles to Users. |
| AAS: UserRoleManager      | AAS: \[user\_roles:create:\*, user\_roles:retrieve:\*, user\_roles:search:\*, user\_roles:delete:\*, | AAS role with permissions to assign Roles to Users, but cannot create delete or modify Users or Roles. |
| HVS: AttestationRegister  | HVS: \[host\_tls\_policies:create:\*, hosts:create:\*, hosts:store:\*, hosts:search:\*, host\_unique\_flavors:create:\*, flavors:search:\*, tpm\_passwords:retrieve:\*, tpm\_passwords:create:\*, host\_aiks:certify:\* | Role used for Trust Agent provisioning. Used to create the installation token provided during installation. |
| HVS: HostSigner           | HVS: host\_signing\_key\_certificates:create:\*              | Used for installation of the Workload Agent                  |
| TA: CustomClaimCreator    | HVS: "host_aiks:certify:*","tpm_endorsements:create:*", "tpm_endorsements:search:*",AAS: "credential:create:*"  | Used for installation of the Trust Agent                  |

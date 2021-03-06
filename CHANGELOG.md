## 0.7.1 (Unreleased)

IMPROVEMENTS:

 * storage/s3: Support `max_parallel` option to limit concurrent outstanding
   requests [GH-2466]
 * storage/s3: Use pooled transport for http client [GH-2481]
 * storage/etcd3: Add `discovery_srv` option to query for SRV records to find
   servers [GH-2521]

BUG FIXES:

 * secret/pki: Don't lowercase O/OU values in certs [GH-2555]
 * storage/consul: Properly handle state events rather than timing out
   [GH-2548]
 * storage/etcd3: Ensure locks are released if client is improperly shut down
   [GH-2526]

## 0.7.0 (March 21th, 2017)

SECURITY:

 * Common name not being validated when `exclude_cn_from_sans` option used in
   `pki` backend: When using a role in the `pki` backend that specified the
   `exclude_cn_from_sans` option, the common name would not then be properly
   validated against the role's constraints. This has been fixed. We recommend
   any users of this feature to upgrade to 0.7 as soon as feasible.

DEPRECATIONS/CHANGES:

 * List Operations Always Use Trailing Slash: Any list operation, whether via
   the `GET` or `LIST` HTTP verb, will now internally canonicalize the path to
   have a trailing slash. This makes policy writing more predictable, as it
   means clients will no longer work or fail based on which client they're
   using or which HTTP verb they're using. However, it also means that policies
   allowing `list` capability must be carefully checked to ensure that they
   contain a trailing slash; some policies may need to be split into multiple
   stanzas to accommodate.
 * PKI Defaults to Unleased Certificates: When issuing certificates from the
   PKI backend, by default, no leases will be issued. If you want to manually
   revoke a certificate, its serial number can be used with the `pki/revoke`
   endpoint. Issuing leases is still possible by enabling the `generate_lease`
   toggle in PKI role entries (this will default to `true` for upgrades, to
   keep existing behavior), which will allow using lease IDs to revoke
   certificates. For installations issuing large numbers of certificates (tens
   to hundreds of thousands, or millions), this will significantly improve
   Vault startup time since leases associated with these certificates will not
   have to be loaded; however note that it also means that revocation of a
   token used to issue certificates will no longer add these certificates to a
   CRL. If this behavior is desired or needed, consider keeping leases enabled
   and ensuring lifetimes are reasonable, and issue long-lived certificates via
   a different role with leases disabled.

FEATURES:

 * **Replication (Enterprise)**: Vault Enterprise now has support for creating
   a multi-datacenter replication set between clusters. The current replication
   offering is based on an asynchronous primary/secondary (1:N) model that
   replicates static data while keeping dynamic data (leases, tokens)
   cluster-local, focusing on horizontal scaling for high-throughput and
   high-fanout deployments.
 * **Response Wrapping & Replication in the Vault Enterprise UI**: Vault
   Enterprise UI now supports looking up and rotating response wrapping tokens,
   as well as creating tokens with arbitrary values inside. It also now
   supports replication functionality, enabling the configuration of a
   replication set in the UI.
 * **Expanded Access Control Policies**: Access control policies can now
   specify allowed and denied parameters -- and, optionally, their values -- to
   control what a client can and cannot submit during an API call. Policies can
   also specify minimum/maximum response wrapping TTLs to both enforce the use
   of response wrapping and control the duration of resultant wrapping tokens.
   See the [policies concepts
   page](https://www.vaultproject.io/docs/concepts/policies.html) for more
   information.
 * **SSH Backend As Certificate Authority**: The SSH backend can now be
   configured to sign host and user certificates. Each mount of the backend
   acts as an independent signing authority. The CA key pair can be configured
   for each mount and the public key is accessible via an unauthenticated API
   call; additionally, the backend can generate a public/private key pair for
   you. We recommend using separate mounts for signing host and user
   certificates.

IMPROVEMENTS:

 * api/request: Passing username and password information in API request
   [GH-2469]
 * audit: Logging the token's use count with authentication response and
   logging the remaining uses of the client token with request [GH-2437]
 * auth/approle: Support for restricting the number of uses on the tokens
   issued [GH-2435]
 * auth/aws-ec2: AWS EC2 auth backend now supports constraints for VPC ID,
   Subnet ID and Region [GH-2407]
 * auth/ldap: Use the value of the `LOGNAME` or `USER` env vars for the
   username if not explicitly set on the command line when authenticating
   [GH-2154]
 * audit: Support adding a configurable prefix (such as `@cee`) before each
   line [GH-2359]
 * core: Canonicalize list operations to use a trailing slash [GH-2390]
 * core: Add option to disable caching on a per-mount level [GH-2455]
 * core: Add ability to require valid client certs in listener config [GH-2457]
 * physical/dynamodb: Implement a session timeout to avoid having to use
   recovery mode in the case of an unclean shutdown, which makes HA much safer
   [GH-2141]
 * secret/pki: O (Organization) values can now be set to role-defined values
   for issued/signed certificates [GH-2369]
 * secret/pki: Certificates issued/signed from PKI backend do not generate
   leases by default [GH-2403]
 * secret/pki: When using DER format, still return the private key type
   [GH-2405]
 * secret/pki: Add an intermediate to the CA chain even if it lacks an
   authority key ID [GH-2465]
 * secret/pki: Add role option to use CSR SANs [GH-2489]
 * secret/ssh: SSH backend as CA to sign user and host certificates [GH-2208]
 * secret/ssh: Support reading of SSH CA public key from `config/ca` endpoint
   and also return it when CA key pair is generated [GH-2483]

BUG FIXES:

 * audit: When auditing headers use case-insensitive comparisons [GH-2362]
 * auth/aws-ec2: Return role period in seconds and not nanoseconds [GH-2374]
 * auth/okta: Fix panic if user had no local groups and/or policies set
   [GH-2367]
 * command/server: Fix parsing of redirect address when port is not mentioned
   [GH-2354]
 * physical/postgresql: Fix listing returning incorrect results if there were
   multiple levels of children [GH-2393]

## 0.6.5 (February 7th, 2017)

FEATURES:

 * **Okta Authentication**: A new Okta authentication backend allows you to use
   Okta usernames and passwords to authenticate to Vault. If provided with an
   appropriate Okta API token, group membership can be queried to assign
   policies; users and groups can be defined locally as well.
 * **RADIUS Authentication**: A new RADIUS authentication backend allows using
   a RADIUS server to authenticate to Vault. Policies can be configured for
   specific users or for any authenticated user.
 * **Exportable Transit Keys**: Keys in `transit` can now be marked as
   `exportable` at creation time. This allows a properly ACL'd user to retrieve
   the associated signing key, encryption key, or HMAC key. The `exportable`
   value is returned on a key policy read and cannot be changed, so if a key is
   marked `exportable` it will always be exportable, and if it is not it will
   never be exportable.
 * **Batch Transit Operations**: `encrypt`, `decrypt` and `rewrap` operations
   in the transit backend now support processing multiple input items in one
   call, returning the output of each item in the response.
 * **Configurable Audited HTTP Headers**: You can now specify headers that you
   want to have included in each audit entry, along with whether each header
   should be HMAC'd or kept plaintext. This can be useful for adding additional
   client or network metadata to the audit logs. 
 * **Transit Backend UI (Enterprise)**: Vault Enterprise UI now supports the transit
   backend, allowing creation, viewing and editing of named keys as well as using
   those keys to perform supported transit operations directly in the UI.
 * **Socket Audit Backend** A new socket audit backend allows audit logs to be sent 
   through TCP, UDP, or UNIX Sockets.

IMPROVEMENTS:

 * auth/aws-ec2: Add support for cross-account auth using STS [GH-2148]
 * auth/aws-ec2: Support issuing periodic tokens [GH-2324]
 * auth/github: Support listing teams and users [GH-2261]
 * auth/ldap: Support adding policies to local users directly, in addition to
   local groups [GH-2152]
 * command/server: Add ability to select and prefer server cipher suites
   [GH-2293]
 * core: Add a nonce to unseal operations as a check (useful mostly for
   support, not as a security principle) [GH-2276]
 * duo: Added ability to supply extra context to Duo pushes [GH-2118]
 * physical/consul: Add option for setting consistency mode on Consul gets
   [GH-2282]
 * physical/etcd: Full v3 API support; code will autodetect which API version
   to use. The v3 code path is significantly less complicated and may be much
   more stable. [GH-2168]
 * secret/pki: Allow specifying OU entries in generated certificate subjects
   [GH-2251]
 * secret mount ui (Enterprise): the secret mount list now shows all mounted
   backends even if the UI cannot browse them. Additional backends can now be
   mounted from the UI as well.

BUG FIXES:

 * auth/token: Fix regression in 0.6.4 where using token store roles as a
   blacklist (with only `disallowed_policies` set) would not work in most
   circumstances [GH-2286]
 * physical/s3: Page responses in client so list doesn't truncate [GH-2224]
 * secret/cassandra: Stop a connection leak that could occur on active node
   failover [GH-2313]
 * secret/pki: When using `sign-verbatim`, don't require a role and use the
   CSR's common name [GH-2243]

## 0.6.4 (December 16, 2016)

SECURITY:

Further details about these security issues can be found in the 0.6.4 upgrade
guide.

 * `default` Policy Privilege Escalation: If a parent token did not have the
   `default` policy attached to its token, it could still create children with
   the `default` policy. This is no longer allowed (unless the parent has
   `sudo` capability for the creation path). In most cases this is low severity
   since the access grants in the `default` policy are meant to be access
   grants that are acceptable for all tokens to have.
 * Leases Not Expired When Limited Use Token Runs Out of Uses: When using
   limited-use tokens to create leased secrets, if the limited-use token was
   revoked due to running out of uses (rather than due to TTL expiration or
   explicit revocation) it would fail to revoke the leased secrets. These
   secrets would still be revoked when their TTL expired, limiting the severity
   of this issue. An endpoint has been added (`auth/token/tidy`) that can
   perform housekeeping tasks on the token store; one of its tasks can detect
   this situation and revoke the associated leases.

FEATURES:

  * **Policy UI (Enterprise)**: Vault Enterprise UI now supports viewing,
    creating, and editing policies.

IMPROVEMENTS:

 * http: Vault now sets a `no-store` cache control header to make it more
   secure in setups that are not end-to-end encrypted [GH-2183]

BUG FIXES:

 * auth/ldap: Don't panic if dialing returns an error and starttls is enabled;
   instead, return the error [GH-2188]
 * ui (Enterprise): Submitting an unseal key now properly resets the
   form so a browser refresh isn't required to continue.

## 0.6.3 (December 6, 2016)

DEPRECATIONS/CHANGES:

 * Request size limitation: A maximum request size of 32MB is imposed to
   prevent a denial of service attack with arbitrarily large requests [GH-2108]
 * LDAP denies passwordless binds by default: In new LDAP mounts, or when
   existing LDAP mounts are rewritten, passwordless binds will be denied by
   default. The new `deny_null_bind` parameter can be set to `false` to allow
   these. [GH-2103]
 * Any audit backend activated satisfies conditions: Previously, when a new
   Vault node was taking over service in an HA cluster, all audit backends were
   required to be loaded successfully to take over active duty. This behavior
   now matches the behavior of the audit logging system itself: at least one
   audit backend must successfully be loaded. The server log contains an error
   when this occurs. This helps keep a Vault HA cluster working when there is a
   misconfiguration on a standby node. [GH-2083]

FEATURES:

 * **Web UI (Enterprise)**: Vault Enterprise now contains a built-in web UI
   that offers access to a number of features, including init/unsealing/sealing,
   authentication via userpass or LDAP, and K/V reading/writing. The capability
   set of the UI will be expanding rapidly in further releases. To enable it,
   set `ui = true` in the top level of Vault's configuration file and point a
   web browser at your Vault address.
 * **Google Cloud Storage Physical Backend**: You can now use GCS for storing
   Vault data [GH-2099]

IMPROVEMENTS:

 * auth/github: Policies can now be assigned to users as well as to teams
   [GH-2079]
 * cli: Set the number of retries on 500 down to 0 by default (no retrying). It
   can be very confusing to users when there is a pause while the retries
   happen if they haven't explicitly set it. With request forwarding the need
   for this is lessened anyways. [GH-2093]
 * core: Response wrapping is now allowed to be specified by backend responses
   (requires backends gaining support) [GH-2088]
 * physical/consul: When announcing service, use the scheme of the Vault server
   rather than the Consul client [GH-2146]
 * secret/consul: Added listing functionality to roles [GH-2065]
 * secret/postgresql: Added `revocation_sql` parameter on the role endpoint to
   enable customization of user revocation SQL statements [GH-2033]
 * secret/transit: Add listing of keys [GH-1987]

BUG FIXES:

 * api/unwrap, command/unwrap: Increase compatibility of `unwrap` command with
   Vault 0.6.1 and older [GH-2014]
 * api/unwrap, command/unwrap: Fix error when no client token exists [GH-2077]
 * auth/approle: Creating the index for the role_id properly [GH-2004]
 * auth/aws-ec2: Handle the case of multiple upgrade attempts when setting the
   instance-profile ARN [GH-2035]
 * auth/ldap: Avoid leaking connections on login [GH-2130]
 * command/path-help: Use the actual error generated by Vault rather than
   always using 500 when there is a path help error [GH-2153]
 * command/ssh: Use temporary file for identity and ensure its deletion before
   the command returns [GH-2016]
 * cli: Fix error printing values with `-field` if the values contained
   formatting directives [GH-2109]
 * command/server: Don't say mlock is supported on OSX when it isn't. [GH-2120]
 * core: Fix bug where a failure to come up as active node (e.g. if an audit
   backend failed) could lead to deadlock [GH-2083]
 * physical/mysql: Fix potential crash during setup due to a query failure
   [GH-2105]
 * secret/consul: Fix panic on user error [GH-2145]

## 0.6.2 (October 5, 2016)

DEPRECATIONS/CHANGES:

 * Convergent Encryption v2: New keys in `transit` using convergent mode will
   use a new nonce derivation mechanism rather than require the user to supply
   a nonce. While not explicitly increasing security, it minimizes the
   likelihood that a user will use the mode improperly and impact the security
   of their keys. Keys in convergent mode that were created in v0.6.1 will
   continue to work with the same mechanism (user-supplied nonce).
 * `etcd` HA off by default: Following in the footsteps of `dynamodb`, the
   `etcd` storage backend now requires that `ha_enabled` be explicitly
   specified in the configuration file. The backend currently has known broken
   HA behavior, so this flag discourages use by default without explicitly
   enabling it. If you are using this functionality, when upgrading, you should
   set `ha_enabled` to `"true"` *before* starting the new versions of Vault.
 * Default/Max lease/token TTLs are now 32 days: In previous versions of Vault
   the default was 30 days, but moving it to 32 days allows some operations
   (e.g. reauthenticating, renewing, etc.) to be performed via a monthly cron
   job.
 * AppRole Secret ID endpoints changed: Secret ID and Secret ID accessors are
   no longer part of request URLs. The GET and DELETE operations are now moved
   to new endpoints (`/lookup` and `/destroy`) which consumes the input from
   the body and not the URL.
 * AppRole requires at least one constraint: previously it was sufficient to
   turn off all AppRole authentication constraints (secret ID, CIDR block) and
   use the role ID only. It is now required that at least one additional
   constraint is enabled. Existing roles are unaffected, but any new roles or
   updated roles will require this.
 * Reading wrapped responses from `cubbyhole/response` is deprecated. The
   `sys/wrapping/unwrap` endpoint should be used instead as it provides
   additional security, auditing, and other benefits. The ability to read
   directly will be removed in a future release.
 * Request Forwarding is now on by default: in 0.6.1 this required toggling on,
   but is now enabled by default. This can be disabled via the
   `"disable_clustering"` parameter in Vault's
   [config](https://www.vaultproject.io/docs/config/index.html), or per-request
   with the `X-Vault-No-Request-Forwarding` header.
 * In prior versions a bug caused the `bound_iam_role_arn` value in the
   `aws-ec2` authentication backend to actually use the instance profile ARN.
   This has been corrected, but as a result there is a behavior change. To
   match using the instance profile ARN, a new parameter
   `bound_iam_instance_profile_arn` has been added. Existing roles will
   automatically transfer the value over to the correct parameter, but the next
   time the role is updated, the new meanings will take effect.

FEATURES:

 * **Secret ID CIDR Restrictions in `AppRole`**: Secret IDs generated under an
   approle can now specify a list of CIDR blocks from where the requests to
   generate secret IDs should originate from. If an approle already has CIDR
   restrictions specified, the CIDR restrictions on the secret ID should be a
   subset of those specified on the role [GH-1910]
 * **Initial Root Token PGP Encryption**: Similar to `generate-root`, the root
   token created at initialization time can now be PGP encrypted [GH-1883]
 * **Support Chained Intermediate CAs in `pki`**: The `pki` backend now allows,
   when a CA cert is being supplied as a signed root or intermediate, a trust
   chain of arbitrary length. The chain is returned as a parameter at
   certificate issue/sign time and is retrievable independently as well.
   [GH-1694]
 * **Response Wrapping Enhancements**: There are new endpoints to look up
   response wrapped token parameters; wrap arbitrary values; rotate wrapping
   tokens; and unwrap with enhanced validation. In addition, list operations
   can now be response-wrapped. [GH-1927]
 * Transit features: The `transit` backend now supports generating random bytes
   and SHA sums; HMACs; and signing and verification functionality using EC
   keys (P-256 curve)

IMPROVEMENTS:

 * api: Return error when an invalid (as opposed to incorrect) unseal key is
   submitted, rather than ignoring it [GH-1782]
 * api: Add method to call `auth/token/create-orphan` endpoint [GH-1834]
 * api: Rekey operation now redirects from standbys to master [GH-1862]
 * audit/file: Sending a `SIGHUP` to Vault now causes Vault to close and
   re-open the log file, making it easier to rotate audit logs [GH-1953]
 * auth/aws-ec2: EC2 instances can get authenticated by presenting the identity
   document and its SHA256 RSA digest [GH-1961]
 * auth/aws-ec2: IAM bound parameters on the aws-ec2 backend will perform a
   prefix match instead of exact match [GH-1943]
 * auth/aws-ec2: Added a new constraint `bound_iam_instance_profile_arn` to
   refer to IAM instance profile ARN and fixed the earlier `bound_iam_role_arn`
   to refer to IAM role ARN instead of the instance profile ARN [GH-1913]
 * auth/aws-ec2: Backend generates the nonce by default and clients can
   explicitly disable reauthentication by setting empty nonce [GH-1889]
 * auth/token: Added warnings if tokens and accessors are used in URLs [GH-1806]
 * command/format: The `format` flag on select CLI commands takes `yml` as an
   alias for `yaml` [GH-1899]
 * core: Allow the size of the read cache to be set via the config file, and
   change the default value to 1MB (from 32KB) [GH-1784]
 * core: Allow single and two-character path parameters for most places
   [GH-1811]
 * core: Allow list operations to be response-wrapped [GH-1814]
 * core: Provide better protection against timing attacks in Shamir code
   [GH-1877]
 * core: Unmounting/disabling backends no longer returns an error if the mount
   didn't exist. This is line with elsewhere in Vault's API where `DELETE` is
   an idempotent operation. [GH-1903]
 * credential/approle: At least one constraint is required to be enabled while
   creating and updating a role [GH-1882]
 * secret/cassandra: Added consistency level for use with roles [GH-1931]
 * secret/mysql: SQL for revoking user can be configured on the role [GH-1914]
 * secret/transit: Use HKDF (RFC 5869) as the key derivation function for new
   keys [GH-1812]
 * secret/transit: Empty plaintext values are now allowed [GH-1874]

BUG FIXES:

 * audit: Fix panic being caused by some values logging as underlying Go types
   instead of formatted strings [GH-1912]
 * auth/approle: Fixed panic on deleting approle that doesn't exist [GH-1920]
 * auth/approle: Not letting secret IDs and secret ID accessors to get logged
   in plaintext in audit logs [GH-1947]
 * auth/aws-ec2: Allow authentication if the underlying host is in a bad state
   but the instance is running [GH-1884]
 * auth/token: Fixed metadata getting missed out from token lookup response by
   gracefully handling token entry upgrade [GH-1924]
 * cli: Don't error on newline in token file [GH-1774]
 * core: Pass back content-type header for forwarded requests [GH-1791]
 * core: Fix panic if the same key was given twice to `generate-root` [GH-1827]
 * core: Fix potential deadlock on unmount/remount [GH-1793]
 * physical/file: Remove empty directories from the `file` storage backend [GH-1821]
 * physical/zookeeper: Remove empty directories from the `zookeeper` storage
   backend and add a fix to the `file` storage backend's logic [GH-1964]
 * secret/aws: Added update operation to `aws/sts` path to consider `ttl`
   parameter [39b75c6]
 * secret/aws: Mark STS secrets as non-renewable [GH-1804]
 * secret/cassandra: Properly store session for re-use [GH-1802]
 * secret/ssh: Fix panic when revoking SSH dynamic keys [GH-1781]

## 0.6.1 (August 22, 2016)

DEPRECATIONS/CHANGES:

 * Once the active node is 0.6.1, standby nodes must also be 0.6.1 in order to
   connect to the HA cluster. We recommend following our [general upgrade
   instructions](https://www.vaultproject.io/docs/install/upgrade.html) in
   addition to 0.6.1-specific upgrade instructions to ensure that this is not
   an issue.
 * Status codes for sealed/uninitialized Vaults have changed to `503`/`501`
   respectively. See the [version-specific upgrade
   guide](https://www.vaultproject.io/docs/install/upgrade-to-0.6.1.html) for
   more details.
 * Root tokens (tokens with the `root` policy) can no longer be created except
   by another root token or the `generate-root` endpoint.
 * Issued certificates from the `pki` backend against new roles created or
   modified after upgrading will contain a set of default key usages.
 * The `dynamodb` physical data store no longer supports HA by default. It has
   some non-ideal behavior around failover that was causing confusion. See the
   [documentation](https://www.vaultproject.io/docs/config/index.html#ha_enabled)
   for information on enabling HA mode. It is very important that this
   configuration is added _before upgrading_.
 * The `ldap` backend no longer searches for `memberOf` groups as part of its
   normal flow. Instead, the desired group filter must be specified. This fixes
   some errors and increases speed for directories with different structures,
   but if this behavior has been relied upon, ensure that you see the upgrade
   notes _before upgrading_.
 * `app-id` is now deprecated with the addition of the new AppRole backend.
   There are no plans to remove it, but we encourage using AppRole whenever
   possible, as it offers enhanced functionality and can accommodate many more
   types of authentication paradigms.

FEATURES:

 * **AppRole Authentication Backend**: The `approle` backend is a
   machine-oriented authentication backend that provides a similar concept to
   App-ID while adding many missing features, including a pull model that
   allows for the backend to generate authentication credentials rather than
   requiring operators or other systems to push credentials in. It should be
   useful in many more situations than App-ID. The inclusion of this backend
   deprecates App-ID. [GH-1426]
 * **Request Forwarding**: Vault servers can now forward requests to each other
   rather than redirecting clients. This feature is off by default in 0.6.1 but
   will be on by default in the next release. See the [HA concepts
   page](https://www.vaultproject.io/docs/concepts/ha.html) for information on
   enabling and configuring it. [GH-443]
 * **Convergent Encryption in `Transit`**: The `transit` backend now supports a
   convergent encryption mode where the same plaintext will produce the same
   ciphertext. Although very useful in some situations, this has potential
   security implications, which are mostly mitigated by requiring the use of
   key derivation when convergent encryption is enabled. See [the `transit`
   backend
   documentation](https://www.vaultproject.io/docs/secrets/transit/index.html)
   for more details. [GH-1537]
 * **Improved LDAP Group Filters**: The `ldap` auth backend now uses templates
   to define group filters, providing the capability to support some
   directories that could not easily be supported before (especially specific
   Active Directory setups with nested groups). [GH-1388]
 * **Key Usage Control in `PKI`**: Issued certificates from roles created or
   modified after upgrading contain a set of default key usages for increased
   compatibility with OpenVPN and some other software. This set can be changed
   when writing a role definition. Existing roles are unaffected. [GH-1552]
 * **Request Retrying in the CLI and Go API**: Requests that fail with a `5xx`
   error code will now retry after a backoff. The maximum total number of
   retries (including disabling this functionality) can be set with an
   environment variable. See the [environment variable
   documentation](https://www.vaultproject.io/docs/commands/environment.html)
   for more details. [GH-1594]
 * **Service Discovery in `vault init`**: The new `-auto` option on `vault init`
   will perform service discovery using Consul. When only one node is discovered,
   it will be initialized and when more than one node is discovered, they will
   be output for easy selection. See `vault init --help` for more details. [GH-1642]
 * **MongoDB Secret Backend**: Generate dynamic unique MongoDB database
   credentials based on configured roles. Sponsored by
   [CommerceHub](http://www.commercehub.com/). [GH-1414]
 * **Circonus Metrics Integration**: Vault can now send metrics to
   [Circonus](http://www.circonus.com/). See the [configuration
   documentation](https://www.vaultproject.io/docs/config/index.html) for
   details. [GH-1646]

IMPROVEMENTS:

 * audit: Added a unique identifier to each request which will also be found in
   the request portion of the response. [GH-1650]
 * auth/aws-ec2: Added a new constraint `bound_account_id` to the role
   [GH-1523]
 * auth/aws-ec2: Added a new constraint `bound_iam_role_arn` to the role
   [GH-1522]
 * auth/aws-ec2: Added `ttl` field for the role [GH-1703]
 * auth/ldap, secret/cassandra, physical/consul: Clients with `tls.Config`
   have the minimum TLS version set to 1.2 by default. This is configurable.
 * auth/token: Added endpoint to list accessors [GH-1676]
 * auth/token: Added `disallowed_policies` option to token store roles [GH-1681]
 * auth/token: `root` or `sudo` tokens can now create periodic tokens via
   `auth/token/create`; additionally, the same token can now be periodic and
   have an explicit max TTL [GH-1725]
 * build: Add support for building on Solaris/Illumos [GH-1726]
 * cli: Output formatting in the presence of warnings in the response object
   [GH-1533]
 * cli: `vault auth` command supports a `-path` option to take in the path at
   which the auth backend is enabled, thereby allowing authenticating against
   different paths using the command options [GH-1532]
 * cli: `vault auth -methods` will now display the config settings of the mount
   [GH-1531]
 * cli: `vault read/write/unwrap -field` now allows selecting token response
   fields [GH-1567]
 * cli: `vault write -field` now allows selecting wrapped response fields
   [GH-1567]
 * command/status: Version information and cluster details added to the output
   of `vault status` command [GH-1671]
 * core: Response wrapping is now enabled for login endpoints [GH-1588]
 * core: The duration of leadership is now exported via events through
   telemetry [GH-1625]
 * core: `sys/capabilities-self` is now accessible as part of the `default`
   policy [GH-1695]
 * core: `sys/renew` is now accessible as part of the `default` policy [GH-1701]
 * core: Unseal keys will now be returned in both hex and base64 forms, and
   either can be used [GH-1734]
 * core: Responses from most `/sys` endpoints now return normal `api.Secret`
   structs in addition to the values they carried before. This means that
   response wrapping can now be used with most authenticated `/sys` operations
   [GH-1699]
 * physical/etcd: Support `ETCD_ADDR` env var for specifying addresses [GH-1576]
 * physical/consul: Allowing additional tags to be added to Consul service
   registration via `service_tags` option [GH-1643]
 * secret/aws: Listing of roles is supported now  [GH-1546]
 * secret/cassandra: Add `connect_timeout` value for Cassandra connection
   configuration [GH-1581]
 * secret/mssql,mysql,postgresql: Reading of connection settings is supported
   in all the sql backends [GH-1515]
 * secret/mysql: Added optional maximum idle connections value to MySQL
   connection configuration [GH-1635]
 * secret/mysql: Use a combination of the role name and token display name in
   generated user names and allow the length to be controlled [GH-1604]
 * secret/{cassandra,mssql,mysql,postgresql}: SQL statements can now be passed
   in via one of four ways: a semicolon-delimited string, a base64-delimited
   string, a serialized JSON string array, or a base64-encoded serialized JSON
   string array [GH-1686]
 * secret/ssh: Added `allowed_roles` to vault-ssh-helper's config and returning
   role name as part of response of `verify` API
 * secret/ssh: Added passthrough of command line arguments to `ssh` [GH-1680]
 * sys/health: Added version information to the response of health status
   endpoint [GH-1647]
 * sys/health: Cluster information isbe returned as part of health status when
   Vault is unsealed [GH-1671]
 * sys/mounts: MountTable data is compressed before serializing to accommodate
   thousands of mounts [GH-1693]
 * website: The [token
   concepts](https://www.vaultproject.io/docs/concepts/tokens.html) page has
   been completely rewritten [GH-1725]

BUG FIXES:

 * auth/aws-ec2: Added a nil check for stored whitelist identity object
   during renewal [GH-1542]
 * auth/cert: Fix panic if no client certificate is supplied [GH-1637]
 * auth/token: Don't report that a non-expiring root token is renewable, as
   attempting to renew it results in an error [GH-1692]
 * cli: Don't retry a command when a redirection is received [GH-1724]
 * core: Fix regression causing status codes to be `400` in most non-5xx error
   cases [GH-1553]
 * core: Fix panic that could occur during a leadership transition [GH-1627]
 * physical/postgres: Remove use of prepared statements as this causes
   connection multiplexing software to break [GH-1548]
 * physical/consul: Multiple Vault nodes on the same machine leading to check ID
   collisions were resulting in incorrect health check responses [GH-1628]
 * physical/consul: Fix deregistration of health checks on exit [GH-1678]
 * secret/postgresql: Check for existence of role before attempting deletion
   [GH-1575]
 * secret/postgresql: Handle revoking roles that have privileges on sequences
   [GH-1573]
 * secret/postgresql(,mysql,mssql): Fix incorrect use of database over
   transaction object which could lead to connection exhaustion [GH-1572]
 * secret/pki: Fix parsing CA bundle containing trailing whitespace [GH-1634]
 * secret/pki: Fix adding email addresses as SANs [GH-1688]
 * secret/pki: Ensure that CRL values are always UTC, per RFC [GH-1727]
 * sys/seal-status: Fixed nil Cluster object while checking seal status [GH-1715]

## 0.6.0 (June 14th, 2016)

SECURITY:

 * Although `sys/revoke-prefix` was intended to revoke prefixes of secrets (via
   lease IDs, which incorporate path information) and
   `auth/token/revoke-prefix` was intended to revoke prefixes of tokens (using
   the tokens' paths and, since 0.5.2, role information), in implementation
   they both behaved exactly the same way since a single component in Vault is
   responsible for managing lifetimes of both, and the type of the tracked
   lifetime was not being checked. The end result was that either endpoint
   could revoke both secret leases and tokens. We consider this a very minor
   security issue as there are a number of mitigating factors: both endpoints
   require `sudo` capability in addition to write capability, preventing
   blanket ACL path globs from providing access; both work by using the prefix
   to revoke as a part of the endpoint path, allowing them to be properly
   ACL'd; and both are intended for emergency scenarios and users should
   already not generally have access to either one. In order to prevent
   confusion, we have simply removed `auth/token/revoke-prefix` in 0.6, and
   `sys/revoke-prefix` will be meant for both leases and tokens instead.

DEPRECATIONS/CHANGES:

 * `auth/token/revoke-prefix` has been removed. See the security notice for
   details. [GH-1280]
 * Vault will now automatically register itself as the `vault` service when
   using the `consul` backend and will perform its own health checks.  See
   the Consul backend documentation for information on how to disable
   auto-registration and service checks.
 * List operations that do not find any keys now return a `404` status code
   rather than an empty response object [GH-1365]
 * CA certificates issued from the `pki` backend no longer have associated
   leases, and any CA certs already issued will ignore revocation requests from
   the lease manager. This is to prevent CA certificates from being revoked
   when the token used to issue the certificate expires; it was not be obvious
   to users that they need to ensure that the token lifetime needed to be at
   least as long as a potentially very long-lived CA cert.

FEATURES:

 * **AWS EC2 Auth Backend**: Provides a secure introduction mechanism for AWS
   EC2 instances allowing automated retrieval of Vault tokens. Unlike most
   Vault authentication backends, this backend does not require first deploying
   or provisioning security-sensitive credentials (tokens, username/password,
   client certificates, etc). Instead, it treats AWS as a Trusted Third Party
   and uses the cryptographically signed dynamic metadata information that
   uniquely represents each EC2 instance. [Vault
   Enterprise](https://www.hashicorp.com/vault.html) customers have access to a
   turnkey client that speaks the backend API and makes access to a Vault token
   easy.
 * **Response Wrapping**: Nearly any response within Vault can now be wrapped
   inside a single-use, time-limited token's cubbyhole, taking the [Cubbyhole
   Authentication
   Principles](https://www.hashicorp.com/blog/vault-cubbyhole-principles.html)
   mechanism to its logical conclusion. Retrieving the original response is as
   simple as a single API command or the new `vault unwrap` command. This makes
   secret distribution easier and more secure, including secure introduction.
 * **Azure Physical Backend**: You can now use Azure blob object storage as
   your Vault physical data store [GH-1266]
 * **Swift Physical Backend**: You can now use Swift blob object storage as
   your Vault physical data store [GH-1425]
 * **Consul Backend Health Checks**: The Consul backend will automatically
   register a `vault` service and perform its own health checking. By default
   the active node can be found at `active.vault.service.consul` and all with
   standby nodes are `standby.vault.service.consul`. Sealed vaults are marked
   critical and are not listed by default in Consul's service discovery.  See
   the documentation for details. [GH-1349]
 * **Explicit Maximum Token TTLs**: You can now set explicit maximum TTLs on
   tokens that do not honor changes in the system- or mount-set values. This is
   useful, for instance, when the max TTL of the system or the `auth/token`
   mount must be set high to accommodate certain needs but you want more
   granular restrictions on tokens being issued directly from the Token
   authentication backend at `auth/token`. [GH-1399]
 * **Non-Renewable Tokens**: When creating tokens directly through the token
   authentication backend, you can now specify in both token store roles and
   the API whether or not a token should be renewable, defaulting to `true`.
 * **RabbitMQ Secret Backend**: Vault can now generate credentials for
   RabbitMQ. Vhosts and tags can be defined within roles. [GH-788]

IMPROVEMENTS:

 * audit: Add the DisplayName value to the copy of the Request object embedded
   in the associated Response, to match the original Request object [GH-1387]
 * audit: Enable auditing of the `seal` and `step-down` commands [GH-1435]
 * backends: Remove most `root`/`sudo` paths in favor of normal ACL mechanisms.
   A particular exception are any current MFA paths. A few paths in `token` and
   `sys` also require `root` or `sudo`. [GH-1478]
 * command/auth: Restore the previous authenticated token if the `auth` command
   fails to authenticate the provided token [GH-1233]
 * command/write: `-format` and `-field` can now be used with the `write`
   command [GH-1228]
 * core: Add `mlock` support for FreeBSD, OpenBSD, and Darwin [GH-1297]
 * core: Don't keep lease timers around when tokens are revoked [GH-1277]
 * core: If using the `disable_cache` option, caches for the policy store and
   the `transit` backend are now disabled as well [GH-1346]
 * credential/cert: Renewal requests are rejected if the set of policies has
   changed since the token was issued [GH-477]
 * credential/cert: Check CRLs for specific non-CA certs configured in the
   backend [GH-1404]
 * credential/ldap: If `groupdn` is not configured, skip searching LDAP and
   only return policies for local groups, plus a warning [GH-1283]
 * credential/ldap: `vault list` support for users and groups [GH-1270]
 * credential/ldap: Support for the `memberOf` attribute for group membership
   searching [GH-1245]
 * credential/userpass: Add list support for users [GH-911]
 * credential/userpass: Remove user configuration paths from requiring sudo, in
   favor of normal ACL mechanisms [GH-1312]
 * credential/token: Sanitize policies and add `default` policies in appropriate
   places [GH-1235]
 * credential/token: Setting the renewable status of a token is now possible
   via `vault token-create` and the API. The default is true, but tokens can be
   specified as non-renewable. [GH-1499]
 * secret/aws: Use chain credentials to allow environment/EC2 instance/shared
   providers [GH-307]
 * secret/aws: Support for STS AssumeRole functionality [GH-1318]
 * secret/consul: Reading consul access configuration supported. The response
   will contain non-sensitive information only [GH-1445]
 * secret/pki: Added `exclude_cn_from_sans` field to prevent adding the CN to
   DNS or Email Subject Alternate Names [GH-1220]
 * secret/pki: Added list support for certificates [GH-1466]
 * sys/capabilities: Enforce ACL checks for requests that query the capabilities
   of a token on a given path [GH-1221]
 * sys/health: Status information can now be retrieved with `HEAD` [GH-1509]

BUG FIXES:

 * command/read: Fix panic when using `-field` with a non-string value [GH-1308]
 * command/token-lookup: Fix TTL showing as 0 depending on how a token was
   created. This only affected the value shown at lookup, not the token
   behavior itself. [GH-1306]
 * command/various: Tell the JSON decoder to not convert all numbers to floats;
   fixes some various places where numbers were showing up in scientific
   notation
 * command/server: Prioritized `devRootTokenID` and `devListenAddress` flags
   over their respective env vars [GH-1480]
 * command/ssh: Provided option to disable host key checking. The automated
   variant of `vault ssh` command uses `sshpass` which was failing to handle
   host key checking presented by the `ssh` binary. [GH-1473]
 * core: Properly persist mount-tuned TTLs for auth backends [GH-1371]
 * core: Don't accidentally crosswire SIGINT to the reload handler [GH-1372]
 * credential/github: Make organization comparison case-insensitive during
   login [GH-1359]
 * credential/github: Fix panic when renewing a token created with some earlier
   versions of Vault [GH-1510]
 * credential/github: The token used to log in via `vault auth` can now be
   specified in the `VAULT_AUTH_GITHUB_TOKEN` environment variable [GH-1511]
 * credential/ldap: Fix problem where certain error conditions when configuring
   or opening LDAP connections would cause a panic instead of return a useful
   error message [GH-1262]
 * credential/token: Fall back to normal parent-token semantics if
   `allowed_policies` is empty for a role. Using `allowed_policies` of
   `default` resulted in the same behavior anyways. [GH-1276]
 * credential/token: Fix issues renewing tokens when using the "suffix"
   capability of token roles [GH-1331]
 * credential/token: Fix lookup via POST showing the request token instead of
   the desired token [GH-1354]
 * credential/various: Fix renewal conditions when `default` policy is not
   contained in the backend config [GH-1256]
 * physical/s3: Don't panic in certain error cases from bad S3 responses [GH-1353]
 * secret/consul: Use non-pooled Consul API client to avoid leaving files open
   [GH-1428]
 * secret/pki: Don't check whether a certificate is destined to be a CA
   certificate if sign-verbatim endpoint is used [GH-1250]

## 0.5.3 (May 27th, 2016)

SECURITY:

 * Consul ACL Token Revocation: An issue was reported to us indicating that
   generated Consul ACL tokens were not being properly revoked. Upon
   investigation, we found that this behavior was reproducible in a specific
   scenario: when a generated lease for a Consul ACL token had been renewed
   prior to revocation. In this case, the generated token was not being
   properly persisted internally through the renewal function, leading to an
   error during revocation due to the missing token. Unfortunately, this was
   coded as a user error rather than an internal error, and the revocation
   logic was expecting internal errors if revocation failed. As a result, the
   revocation logic believed the revocation to have succeeded when it in fact
   failed, causing the lease to be dropped while the token was still valid
   within Consul. In this release, the Consul backend properly persists the
   token through renewals, and the revocation logic has been changed to
   consider any error type to have been a failure to revoke, causing the lease
   to persist and attempt to be revoked later.

We have written an example shell script that searches through Consul's ACL
tokens and looks for those generated by Vault, which can be used as a template
for a revocation script as deemed necessary for any particular security
response. The script is available at
https://gist.github.com/jefferai/6233c2963f9407a858d84f9c27d725c0

Please note that any outstanding leases for Consul tokens produced prior to
0.5.3 that have been renewed will continue to exhibit this behavior. As a
result, we recommend either revoking all tokens produced by the backend and
issuing new ones, or if needed, a more advanced variant of the provided example
could use the timestamp embedded in each generated token's name to decide which
tokens are too old and should be deleted. This could then be run periodically
up until the maximum lease time for any outstanding pre-0.5.3 tokens has
expired.

This is a security-only release. There are no other code changes since 0.5.2.
The binaries have one additional change: they are built against Go 1.6.1 rather
than Go 1.6, as Go 1.6.1 contains two security fixes to the Go programming
language itself.

## 0.5.2 (March 16th, 2016)

FEATURES:

 * **MSSQL Backend**: Generate dynamic unique MSSQL database credentials based
   on configured roles [GH-998]
 * **Token Accessors**: Vault now provides an accessor with each issued token.
   This accessor is an identifier that can be used for a limited set of
   actions, notably for token revocation. This value can be logged in
   plaintext to audit logs, and in combination with the plaintext metadata
   logged to audit logs, provides a searchable and straightforward way to
   revoke particular users' or services' tokens in many cases. To enable
   plaintext audit logging of these accessors, set `hmac_accessor=false` when
   enabling an audit backend.
 * **Token Credential Backend Roles**: Roles can now be created in the `token`
   credential backend that allow modifying token behavior in ways that are not
   otherwise exposed or easily delegated. This allows creating tokens with a
   fixed set (or subset) of policies (rather than a subset of the calling
   token's), periodic tokens with a fixed TTL but no expiration, specified
   prefixes, and orphans.
 * **Listener Certificate Reloading**: Vault's configured listeners now reload
   their TLS certificate and private key when the Vault process receives a
   SIGHUP.

IMPROVEMENTS:

 * auth/token: Endpoints optionally accept tokens from the HTTP body rather
   than just from the URLs [GH-1211]
 * auth/token,sys/capabilities: Added new endpoints
   `auth/token/lookup-accessor`, `auth/token/revoke-accessor` and
   `sys/capabilities-accessor`, which enables performing the respective actions
   with just the accessor of the tokens, without having access to the actual
   token [GH-1188]
 * core: Ignore leading `/` in policy paths [GH-1170]
 * core: Ignore leading `/` in mount paths [GH-1172]
 * command/policy-write: Provided HCL is now validated for format violations
   and provides helpful information around where the violation occurred
   [GH-1200]
 * command/server: The initial root token ID when running in `-dev` mode can
   now be specified via `-dev-root-token-id` or the environment variable
   `VAULT_DEV_ROOT_TOKEN_ID` [GH-1162]
 * command/server: The listen address when running in `-dev` mode can now be
   specified via `-dev-listen-address` or the environment variable
   `VAULT_DEV_LISTEN_ADDRESS` [GH-1169]
 * command/server: The configured listeners now reload their TLS
   certificates/keys when Vault is SIGHUP'd [GH-1196]
 * command/step-down: New `vault step-down` command and API endpoint to force
   the targeted node to give up active status, but without sealing. The node
   will wait ten seconds before attempting to grab the lock again. [GH-1146]
 * command/token-renew: Allow no token to be passed in; use `renew-self` in
   this case. Change the behavior for any token being passed in to use `renew`.
   [GH-1150]
 * credential/app-id: Allow `app-id` parameter to be given in the login path;
   this causes the `app-id` to be part of the token path, making it easier to
   use with `revoke-prefix` [GH-424]
 * credential/cert: Non-CA certificates can be used for authentication. They
   must be matched exactly (issuer and serial number) for authentication, and
   the certificate must carry the client authentication or 'any' extended usage
   attributes. [GH-1153]
 * credential/cert: Subject and Authority key IDs are output in metadata; this
   allows more flexible searching/revocation in the audit logs [GH-1183]
 * credential/cert: Support listing configured certs [GH-1212]
 * credential/userpass: Add support for `create`/`update` capability
   distinction in user path, and add user-specific endpoints to allow changing
   the password and policies [GH-1216]
 * credential/token: Add roles [GH-1155]
 * secret/mssql: Add MSSQL backend [GH-998]
 * secret/pki: Add revocation time (zero or Unix epoch) to `pki/cert/SERIAL`
   endpoint [GH-1180]
 * secret/pki: Sanitize serial number in `pki/revoke` endpoint to allow some
   other formats [GH-1187]
 * secret/ssh: Added documentation for `ssh/config/zeroaddress` endpoint.
   [GH-1154]
 * sys: Added new endpoints `sys/capabilities` and `sys/capabilities-self` to
   fetch the capabilities of a token on a given path [GH-1171]
 * sys: Added `sys/revoke-force`, which enables a user to ignore backend errors
   when revoking a lease, necessary in some emergency/failure scenarios
   [GH-1168]
 * sys: The return codes from `sys/health` can now be user-specified via query
   parameters [GH-1199]

BUG FIXES:

 * logical/cassandra: Apply hyphen/underscore replacement to the entire
   generated username, not just the UUID, in order to handle token display name
   hyphens [GH-1140]
 * physical/etcd: Output actual error when cluster sync fails [GH-1141]
 * vault/expiration: Not letting the error responses from the backends to skip
   during renewals [GH-1176]

## 0.5.1 (February 25th, 2016)

DEPRECATIONS/CHANGES:

 * RSA keys less than 2048 bits are no longer supported in the PKI backend.
   1024-bit keys are considered unsafe and are disallowed in the Internet PKI.
   The `pki` backend has enforced SHA256 hashes in signatures from the
   beginning, and software that can handle these hashes should be able to
   handle larger key sizes. [GH-1095]
 * The PKI backend now does not automatically delete expired certificates,
   including from the CRL. Doing so could lead to a situation where a time
   mismatch between the Vault server and clients could result in a certificate
   that would not be considered expired by a client being removed from the CRL.
   The new `pki/tidy` endpoint can be used to trigger expirations. [GH-1129]
 * The `cert` backend now performs a variant of channel binding at renewal time
   for increased security. In order to not overly burden clients, a notion of
   identity is used. This functionality can be disabled. See the 0.5.1 upgrade
   guide for more specific information [GH-1127]

FEATURES:

 * **Codebase Audit**: Vault's 0.5 codebase was audited by iSEC. (The terms of
   the audit contract do not allow us to make the results public.) [GH-220]

IMPROVEMENTS:

 * api: The `VAULT_TLS_SERVER_NAME` environment variable can be used to control
   the SNI header during TLS connections [GH-1131]
 * api/health: Add the server's time in UTC to health responses [GH-1117]
 * command/rekey and command/generate-root: These now return the status at
   attempt initialization time, rather than requiring a separate fetch for the
   nonce [GH-1054]
 * credential/cert: Don't require root/sudo tokens for the `certs/` and `crls/`
   paths; use normal ACL behavior instead [GH-468]
 * credential/github: The validity of the token used for login will be checked
   at renewal time [GH-1047]
 * credential/github: The `config` endpoint no longer requires a root token;
   normal ACL path matching applies
 * deps: Use the standardized Go 1.6 vendoring system
 * secret/aws: Inform users of AWS-imposed policy restrictions around STS
   tokens if they attempt to use an invalid policy [GH-1113]
 * secret/mysql: The MySQL backend now allows disabling verification of the
   `connection_url` [GH-1096]
 * secret/pki: Submitted CSRs are now verified to have the correct key type and
   minimum number of bits according to the role. The exception is intermediate
   CA signing and the `sign-verbatim` path [GH-1104]
 * secret/pki: New `tidy` endpoint to allow expunging expired certificates.
   [GH-1129]
 * secret/postgresql: The PostgreSQL backend now allows disabling verification
   of the `connection_url` [GH-1096]
 * secret/ssh: When verifying an OTP, return 400 if it is not valid instead of
   204 [GH-1086]
 * credential/app-id: App ID backend will check the validity of app-id and user-id
   during renewal time [GH-1039]
 * credential/cert: TLS Certificates backend, during renewal, will now match the
   client identity with the client identity used during login [GH-1127]

BUG FIXES:

 * credential/ldap: Properly escape values being provided to search filters
   [GH-1100]
 * secret/aws: Capping on length of usernames for both IAM and STS types
   [GH-1102]
 * secret/pki: If a cert is not found during lookup of a serial number,
   respond with a 400 rather than a 500 [GH-1085]
 * secret/postgresql: Add extra revocation statements to better handle more
   permission scenarios [GH-1053]
 * secret/postgresql: Make connection_url work properly [GH-1112]

## 0.5.0 (February 10, 2016)

SECURITY:

 * Previous versions of Vault could allow a malicious user to hijack the rekey
   operation by canceling an operation in progress and starting a new one. The
   practical application of this is very small. If the user was an unseal key
   owner, they could attempt to do this in order to either receive unencrypted
   reseal keys or to replace the PGP keys used for encryption with ones under
   their control. However, since this would invalidate any rekey progress, they
   would need other unseal key holders to resubmit, which would be rather
   suspicious during this manual operation if they were not also the original
   initiator of the rekey attempt. If the user was not an unseal key holder,
   there is no benefit to be gained; the only outcome that could be attempted
   would be a denial of service against a legitimate rekey operation by sending
   cancel requests over and over. Thanks to Josh Snyder for the report!

DEPRECATIONS/CHANGES:

 * `s3` physical backend: Environment variables are now preferred over
   configuration values. This makes it behave similar to the rest of Vault,
   which, in increasing order of preference, uses values from the configuration
   file, environment variables, and CLI flags. [GH-871]
 * `etcd` physical backend: `sync` functionality is now supported and turned on
   by default. This can be disabled. [GH-921]
 * `transit`: If a client attempts to encrypt a value with a key that does not
   yet exist, what happens now depends on the capabilities set in the client's
   ACL policies. If the client has `create` (or `create` and `update`)
   capability, the key will upsert as in the past. If the client has `update`
   capability, they will receive an error. [GH-1012]
 * `token-renew` CLI command: If the token given for renewal is the same as the
   client token, the `renew-self` endpoint will be used in the API. Given that
   the `default` policy (by default) allows all clients access to the
   `renew-self` endpoint, this makes it much more likely that the intended
   operation will be successful. [GH-894]
 * Token `lookup`: the `ttl` value in the response now reflects the actual
   remaining TTL rather than the original TTL specified when the token was
   created; this value is now located in `creation_ttl` [GH-986]
 * Vault no longer uses grace periods on leases or token TTLs. Uncertainty
   about the length grace period for any given backend could cause confusion
   and uncertainty. [GH-1002]
 * `rekey`: Rekey now requires a nonce to be supplied with key shares. This
   nonce is generated at the start of a rekey attempt and is unique for that
   attempt.
 * `status`: The exit code for the `status` CLI command is now `2` for an
   uninitialized Vault instead of `1`. `1` is returned for errors. This better
   matches the rest of the CLI.

FEATURES:

 * **Split Data/High Availability Physical Backends**: You can now configure
   two separate physical backends: one to be used for High Availability
   coordination and another to be used for encrypted data storage. See the
   [configuration
   documentation](https://vaultproject.io/docs/config/index.html) for details.
   [GH-395]
 * **Fine-Grained Access Control**: Policies can now use the `capabilities` set
   to specify fine-grained control over operations allowed on a path, including
   separation of `sudo` privileges from other privileges. These can be mixed
   and matched in any way desired. The `policy` value is kept for backwards
   compatibility. See the [updated policy
   documentation](https://vaultproject.io/docs/concepts/policies.html) for
   details. [GH-914]
 * **List Support**: Listing is now supported via the API and the new `vault
   list` command. This currently supports listing keys in the `generic` and
   `cubbyhole` backends and a few other places (noted in the IMPROVEMENTS
   section below). Different parts of the API and backends will need to
   implement list capabilities in ways that make sense to particular endpoints,
   so further support will appear over time. [GH-617]
 * **Root Token Generation via Unseal Keys**: You can now use the
   `generate-root` CLI command to generate new orphaned, non-expiring root
   tokens in case the original is lost or revoked (accidentally or
   purposefully). This requires a quorum of unseal key holders. The output
   value is protected via any PGP key of the initiator's choosing or a one-time
   pad known only to the initiator (a suitable pad can be generated via the
   `-genotp` flag to the command. [GH-915]
 * **Unseal Key Archiving**: You can now optionally have Vault store your
   unseal keys in your chosen physical store for disaster recovery purposes.
   This option is only available when the keys are encrypted with PGP. [GH-907]
 * **Keybase Support for PGP Encryption Keys**: You can now specify Keybase
   users when passing in PGP keys to the `init`, `rekey`, and `generate-root`
   CLI commands.  Public keys for these users will be fetched automatically.
   [GH-901]
 * **DynamoDB HA Physical Backend**: There is now a new, community-supported
   HA-enabled physical backend using Amazon DynamoDB. See the [configuration
   documentation](https://vaultproject.io/docs/config/index.html) for details.
   [GH-878]
 * **PostgreSQL Physical Backend**: There is now a new, community-supported
   physical backend using PostgreSQL. See the [configuration
   documentation](https://vaultproject.io/docs/config/index.html) for details.
   [GH-945]
 * **STS Support in AWS Secret Backend**: You can now use the AWS secret
   backend to fetch STS tokens rather than IAM users. [GH-927]
 * **Speedups in the transit backend**: The `transit` backend has gained a
   cache, and now loads only the working set of keys (e.g. from the
   `min_decryption_version` to the current key version) into its working set.
   This provides large speedups and potential memory savings when the `rotate`
   feature of the backend is used heavily.

IMPROVEMENTS:

 * cli: Output secrets sorted by key name [GH-830]
 * cli: Support YAML as an output format [GH-832]
 * cli: Show an error if the output format is incorrect, rather than falling
   back to an empty table [GH-849]
 * cli: Allow setting the `advertise_addr` for HA via the
   `VAULT_ADVERTISE_ADDR` environment variable [GH-581]
 * cli/generate-root: Add generate-root and associated functionality [GH-915]
 * cli/init: Add `-check` flag that returns whether Vault is initialized
   [GH-949]
 * cli/server: Use internal functions for the token-helper rather than shelling
   out, which fixes some problems with using a static binary in Docker or paths
   with multiple spaces when launching in `-dev` mode [GH-850]
 * cli/token-lookup: Add token-lookup command [GH-892]
 * command/{init,rekey}: Allow ASCII-armored keychain files to be arguments for
   `-pgp-keys` [GH-940]
 * conf: Use normal bool values rather than empty/non-empty for the
   `tls_disable` option [GH-802]
 * credential/ldap: Add support for binding, both anonymously (to discover a
   user DN) and via a username and password [GH-975]
 * credential/token: Add `last_renewal_time` to token lookup calls [GH-896]
 * credential/token: Change `ttl` to reflect the current remaining TTL; the
   original value is in `creation_ttl` [GH-1007]
 * helper/certutil: Add ability to parse PKCS#8 bundles [GH-829]
 * logical/aws: You can now get STS tokens instead of IAM users [GH-927]
 * logical/cassandra: Add `protocol_version` parameter to set the CQL proto
   version [GH-1005]
 * logical/cubbyhole: Add cubbyhole access to default policy [GH-936]
 * logical/mysql: Add list support for roles path [GH-984]
 * logical/pki: Fix up key usages being specified for CAs [GH-989]
 * logical/pki: Add list support for roles path [GH-985]
 * logical/pki: Allow `pem_bundle` to be specified as the format, which
   provides a concatenated PEM bundle of returned values [GH-1008]
 * logical/pki: Add 30 seconds of slack to the validity start period to
   accommodate some clock skew in machines [GH-1036]
 * logical/postgres: Add `max_idle_connections` parameter [GH-950]
 * logical/postgres: Add list support for roles path
 * logical/ssh: Add list support for roles path [GH-983]
 * logical/transit: Keys are archived and only keys between the latest version
   and `min_decryption_version` are loaded into the working set. This can
   provide a very large speed increase when rotating keys very often. [GH-977]
 * logical/transit: Keys are now cached, which should provide a large speedup
   in most cases [GH-979]
 * physical/cache: Use 2Q cache instead of straight LRU [GH-908]
 * physical/etcd: Support basic auth [GH-859]
 * physical/etcd: Support sync functionality and enable by default [GH-921]

BUG FIXES:

 * api: Correct the HTTP verb used in the LookupSelf method [GH-887]
 * api: Fix the output of `Sys().MountConfig(...)` to return proper values
   [GH-1017]
 * command/read: Fix panic when an empty argument was given [GH-923]
 * command/ssh: Fix panic when username lookup fails [GH-886]
 * core: When running in standalone mode, don't advertise that we are active
   until post-unseal setup completes [GH-872]
 * core: Update go-cleanhttp dependency to ensure idle connections aren't
   leaked [GH-867]
 * core: Don't allow tokens to have duplicate policies [GH-897]
 * core: Fix regression in `sys/renew` that caused information stored in the
   Secret part of the response to be lost [GH-912]
 * physical: Use square brackets when setting an IPv6-based advertise address
   as the auto-detected advertise address [GH-883]
 * physical/s3: Use an initialized client when using IAM roles to fix a
   regression introduced against newer versions of the AWS Go SDK [GH-836]
 * secret/pki: Fix a condition where unmounting could fail if the CA
   certificate was not properly loaded [GH-946]
 * secret/ssh: Fix a problem where SSH connections were not always closed
   properly [GH-942]

MISC:

 * Clarified our stance on support for community-derived physical backends.
   See the [configuration
   documentation](https://vaultproject.io/docs/config/index.html) for details.
 * Add `vault-java` to libraries [GH-851]
 * Various minor documentation fixes and improvements [GH-839] [GH-854]
   [GH-861] [GH-876] [GH-899] [GH-900] [GH-904] [GH-923] [GH-924] [GH-958]
   [GH-959] [GH-981] [GH-990] [GH-1024] [GH-1025]

BUILD NOTE:

 * The HashiCorp-provided binary release of Vault 0.5.0 is built against a
   patched version of Go 1.5.3 containing two specific bug fixes affecting TLS
   certificate handling. These fixes are in the Go 1.6 tree and were
   cherry-picked on top of stock Go 1.5.3. If you want to examine the way in
   which the releases were built, please look at our [cross-compilation
   Dockerfile](https://github.com/hashicorp/vault/blob/v0.5.0/scripts/cross/Dockerfile-patched-1.5.3).

## 0.4.1 (January 13, 2016)

SECURITY:

  * Build against Go 1.5.3 to mitigate a security vulnerability introduced in
    Go 1.5. For more information, please see
    https://groups.google.com/forum/#!topic/golang-dev/MEATuOi_ei4

This is a security-only release; other than the version number and building
against Go 1.5.3, there are no changes from 0.4.0.

## 0.4.0 (December 10, 2015)

DEPRECATIONS/CHANGES:

 * Policy Name Casing: Policy names are now normalized to lower-case on write,
   helping prevent accidental case mismatches. For backwards compatibility,
   policy names are not currently normalized when reading or deleting. [GH-676]
 * Default etcd port number: the default connection string for the `etcd`
   physical store uses port 2379 instead of port 4001, which is the port used
   by the supported version 2.x of etcd. [GH-753]
 * As noted below in the FEATURES section, if your Vault installation contains
   a policy called `default`, new tokens created will inherit this policy
   automatically.
 * In the PKI backend there have been a few minor breaking changes:
   * The token display name is no longer a valid option for providing a base
   domain for issuance. Since this name is prepended with the name of the
   authentication backend that issued it, it provided a faulty use-case at best
   and a confusing experience at worst. We hope to figure out a better
   per-token value in a future release.
   * The `allowed_base_domain` parameter has been changed to `allowed_domains`,
   which accepts a comma-separated list of domains. This allows issuing
   certificates with DNS subjects across multiple domains. If you had a
   configured `allowed_base_domain` parameter, it will be migrated
   automatically when the role is read (either via a normal read, or via
   issuing a certificate).

FEATURES:

 * **Significantly Enhanced PKI Backend**: The `pki` backend can now generate
   and sign root CA certificates and intermediate CA CSRs. It can also now sign
   submitted client CSRs, as well as a significant number of other
   enhancements. See the updated documentation for the full API. [GH-666]
 * **CRL Checking for Certificate Authentication**: The `cert` backend now
   supports pushing CRLs into the mount and using the contained serial numbers
   for revocation checking. See the documentation for the `cert` backend for
   more info. [GH-330]
 * **Default Policy**: Vault now ensures that a policy named `default` is added
   to every token. This policy cannot be deleted, but it can be modified
   (including to an empty policy). There are three endpoints allowed in the
   default `default` policy, related to token self-management: `lookup-self`,
   which allows a token to retrieve its own information, and `revoke-self` and
   `renew-self`, which are self-explanatory. If your existing Vault
   installation contains a policy called `default`, it will not be overridden,
   but it will be added to each new token created. You can override this
   behavior when using manual token creation (i.e. not via an authentication
   backend) by setting the "no_default_policy" flag to true. [GH-732]

IMPROVEMENTS:

 * api: API client now uses a 60 second timeout instead of indefinite [GH-681]
 * api: Implement LookupSelf, RenewSelf, and RevokeSelf functions for auth
   tokens [GH-739]
 * api: Standardize environment variable reading logic inside the API; the CLI
   now uses this but can still override via command-line parameters [GH-618]
 * audit: HMAC-SHA256'd client tokens are now stored with each request entry.
   Previously they were only displayed at creation time; this allows much
   better traceability of client actions. [GH-713]
 * audit: There is now a `sys/audit-hash` endpoint that can be used to generate
   an HMAC-SHA256'd value from provided data using the given audit backend's
   salt [GH-784]
 * core: The physical storage read cache can now be disabled via
   "disable_cache" [GH-674]
 * core: The unsealing process can now be reset midway through (this feature
   was documented before, but not enabled) [GH-695]
 * core: Tokens can now renew themselves [GH-455]
 * core: Base64-encoded PGP keys can be used with the CLI for `init` and
   `rekey` operations [GH-653]
 * core: Print version on startup [GH-765]
 * core: Access to `sys/policy` and `sys/mounts` now uses the normal ACL system
   instead of requiring a root token [GH-769]
 * credential/token: Display whether or not a token is an orphan in the output
   of a lookup call [GH-766]
 * logical: Allow `.` in path-based variables in many more locations [GH-244]
 * logical: Responses now contain a "warnings" key containing a list of
   warnings returned from the server. These are conditions that did not require
   failing an operation, but of which the client should be aware. [GH-676]
 * physical/(consul,etcd): Consul and etcd now use a connection pool to limit
   the number of outstanding operations, improving behavior when a lot of
   operations must happen at once [GH-677] [GH-780]
 * physical/consul: The `datacenter` parameter was removed; It could not be
   effective unless the Vault node (or the Consul node it was connecting to)
   was in the datacenter specified, in which case it wasn't needed [GH-816]
 * physical/etcd: Support TLS-encrypted connections and use a connection pool
   to limit the number of outstanding operations [GH-780]
 * physical/s3: The S3 endpoint can now be configured, allowing using
   S3-API-compatible storage solutions [GH-750]
 * physical/s3: The S3 bucket can now be configured with the `AWS_S3_BUCKET`
   environment variable [GH-758]
 * secret/consul: Management tokens can now be created [GH-714]

BUG FIXES:

 * api: API client now checks for a 301 response for redirects. Vault doesn't
   generate these, but in certain conditions Go's internal HTTP handler can
   generate them, leading to client errors.
 * cli: `token-create` now supports the `ttl` parameter in addition to the
   deprecated `lease` parameter. [GH-688]
 * core: Return data from `generic` backends on the last use of a limited-use
   token [GH-615]
 * core: Fix upgrade path for leases created in `generic` prior to 0.3 [GH-673]
 * core: Stale leader entries will now be reaped [GH-679]
 * core: Using `mount-tune` on the auth/token path did not take effect.
   [GH-688]
 * core: Fix a potential race condition when (un)sealing the vault with metrics
   enabled [GH-694]
 * core: Fix an error that could happen in some failure scenarios where Vault
   could fail to revert to a clean state [GH-733]
 * core: Ensure secondary indexes are removed when a lease is expired [GH-749]
 * core: Ensure rollback manager uses an up-to-date mounts table [GH-771]
 * everywhere: Don't use http.DefaultClient, as it shares state implicitly and
   is a source of hard-to-track-down bugs [GH-700]
 * credential/token: Allow creating orphan tokens via an API path [GH-748]
 * secret/generic: Validate given duration at write time, not just read time;
   if stored durations are not parseable, return a warning and the default
   duration rather than an error [GH-718]
 * secret/generic: Return 400 instead of 500 when `generic` backend is written
   to with no data fields [GH-825]
 * secret/postgresql: Revoke permissions before dropping a user or revocation
   may fail [GH-699]

MISC:

 * Various documentation fixes and improvements [GH-685] [GH-688] [GH-697]
   [GH-710] [GH-715] [GH-831]

## 0.3.1 (October 6, 2015)

SECURITY:

 * core: In certain failure scenarios, the full values of requests and
   responses would be logged [GH-665]

FEATURES:

 * **Settable Maximum Open Connections**: The `mysql` and `postgresql` backends
   now allow setting the number of maximum open connections to the database,
   which was previously capped to 2. [GH-661]
 * **Renewable Tokens for GitHub**: The `github` backend now supports
   specifying a TTL, enabling renewable tokens. [GH-664]

BUG FIXES:

 * dist: linux-amd64 distribution was dynamically linked [GH-656]
 * credential/github: Fix acceptance tests [GH-651]

MISC:

 * Various minor documentation fixes and improvements [GH-649] [GH-650]
   [GH-654] [GH-663]

## 0.3.0 (September 28, 2015)

DEPRECATIONS/CHANGES:

Note: deprecations and breaking changes in upcoming releases are announced
ahead of time on the "vault-tool" mailing list.

 * **Cookie Authentication Removed**: As of 0.3 the only way to authenticate is
   via the X-Vault-Token header. Cookie authentication was hard to properly
   test, could result in browsers/tools/applications saving tokens in plaintext
   on disk, and other issues. [GH-564]
 * **Terminology/Field Names**: Vault is transitioning from overloading the
   term "lease" to mean both "a set of metadata" and "the amount of time the
   metadata is valid". The latter is now being referred to as TTL (or
   "lease_duration" for backwards-compatibility); some parts of Vault have
   already switched to using "ttl" and others will follow in upcoming releases.
   In particular, the "token", "generic", and "pki" backends accept both "ttl"
   and "lease" but in 0.4 only "ttl" will be accepted. [GH-528]
 * **Downgrade Not Supported**: Due to enhancements in the storage subsystem,
   values written by Vault 0.3+ will not be able to be read by prior versions
   of Vault. There are no expected upgrade issues, however, as with all
   critical infrastructure it is recommended to back up Vault's physical
   storage before upgrading.

FEATURES:

 * **SSH Backend**: Vault can now be used to delegate SSH access to machines,
   via a (recommended) One-Time Password approach or by issuing dynamic keys.
   [GH-385]
 * **Cubbyhole Backend**: This backend works similarly to the "generic" backend
   but provides a per-token workspace. This enables some additional
   authentication workflows (especially for containers) and can be useful to
   applications to e.g. store local credentials while being restarted or
   upgraded, rather than persisting to disk. [GH-612]
 * **Transit Backend Improvements**: The transit backend now allows key
   rotation and datakey generation. For rotation, data encrypted with previous
   versions of the keys can still be decrypted, down to a (configurable)
   minimum previous version; there is a rewrap function for manual upgrades of
   ciphertext to newer versions. Additionally, the backend now allows
   generating and returning high-entropy keys of a configurable bitsize
   suitable for AES and other functions; this is returned wrapped by a named
   key, or optionally both wrapped and plaintext for immediate use. [GH-626]
 * **Global and Per-Mount Default/Max TTL Support**: You can now set the
   default and maximum Time To Live for leases both globally and per-mount.
   Per-mount settings override global settings. Not all backends honor these
   settings yet, but the maximum is a hard limit enforced outside the backend.
   See the documentation for "/sys/mounts/" for details on configuring
   per-mount TTLs.  [GH-469]
 * **PGP Encryption for Unseal Keys**: When initializing or rotating Vault's
   master key, PGP/GPG public keys can now be provided. The output keys will be
   encrypted with the given keys, in order. [GH-570]
 * **Duo Multifactor Authentication Support**: Backends that support MFA can
   now use Duo as the mechanism. [GH-464]
 * **Performance Improvements**: Users of the "generic" backend will see a
   significant performance improvement as the backend no longer creates leases,
   although it does return TTLs (global/mount default, or set per-item) as
   before.  [GH-631]
 * **Codebase Audit**: Vault's codebase was audited by iSEC. (The terms of the
   audit contract do not allow us to make the results public.) [GH-220]

IMPROVEMENTS:

 * audit: Log entries now contain a time field [GH-495]
 * audit: Obfuscated audit entries now use hmac-sha256 instead of sha1 [GH-627]
 * backends: Add ability for a cleanup function to be called on backend unmount
   [GH-608]
 * config: Allow specifying minimum acceptable TLS version [GH-447]
 * core: If trying to mount in a location that is already mounted, be more
   helpful about the error [GH-510]
 * core: Be more explicit on failure if the issue is invalid JSON [GH-553]
 * core: Tokens can now revoke themselves [GH-620]
 * credential/app-id: Give a more specific error when sending a duplicate POST
   to sys/auth/app-id [GH-392]
 * credential/github: Support custom API endpoints (e.g. for Github Enterprise)
   [GH-572]
 * credential/ldap: Add per-user policies and option to login with
   userPrincipalName [GH-420]
 * credential/token: Allow root tokens to specify the ID of a token being
   created from CLI [GH-502]
 * credential/userpass: Enable renewals for login tokens [GH-623]
 * scripts: Use /usr/bin/env to find Bash instead of hardcoding [GH-446]
 * scripts: Use godep for build scripts to use same environment as tests
   [GH-404]
 * secret/mysql: Allow reading configuration data [GH-529]
 * secret/pki: Split "allow_any_name" logic to that and "enforce_hostnames", to
   allow for non-hostname values (e.g. for client certificates) [GH-555]
 * storage/consul: Allow specifying certificates used to talk to Consul
   [GH-384]
 * storage/mysql: Allow SSL encrypted connections [GH-439]
 * storage/s3: Allow using temporary security credentials [GH-433]
 * telemetry: Put telemetry object in configuration to allow more flexibility
   [GH-419]
 * testing: Disable mlock for testing of logical backends so as not to require
   root [GH-479]

BUG FIXES:

 * audit/file: Do not enable auditing if file permissions are invalid [GH-550]
 * backends: Allow hyphens in endpoint patterns (fixes AWS and others) [GH-559]
 * cli: Fixed missing setup of client TLS certificates if no custom CA was
   provided
 * cli/read: Do not include a carriage return when using raw field output
   [GH-624]
 * core: Bad input data could lead to a panic for that session, rather than
   returning an error [GH-503]
 * core: Allow SHA2-384/SHA2-512 hashed certificates [GH-448]
 * core: Do not return a Secret if there are no uses left on a token (since it
   will be unable to be used) [GH-615]
 * core: Code paths that called lookup-self would decrement num_uses and
   potentially immediately revoke a token [GH-552]
 * core: Some /sys/ paths would not properly redirect from a standby to the
   leader [GH-499] [GH-551]
 * credential/aws: Translate spaces in a token's display name to avoid making
   IAM unhappy [GH-567]
 * credential/github: Integration failed if more than ten organizations or
   teams [GH-489]
 * credential/token: Tokens with sudo access to "auth/token/create" can now use
   root-only options [GH-629]
 * secret/cassandra: Work around backwards-incompatible change made in
   Cassandra 2.2 preventing Vault from properly setting/revoking leases
   [GH-549]
 * secret/mysql: Use varbinary instead of varchar to avoid InnoDB/UTF-8 issues
   [GH-522]
 * secret/postgres: Explicitly set timezone in connections [GH-597]
 * storage/etcd: Renew semaphore periodically to prevent leadership flapping
   [GH-606]
 * storage/zk: Fix collisions in storage that could lead to data unavailability
   [GH-411]

MISC:

 * Various documentation fixes and improvements [GH-412] [GH-474] [GH-476]
   [GH-482] [GH-483] [GH-486] [GH-508] [GH-568] [GH-574] [GH-586] [GH-590]
   [GH-591] [GH-592] [GH-595] [GH-613] [GH-637]
 * Less "armon" in stack traces [GH-453]
 * Sourcegraph integration [GH-456]

## 0.2.0 (July 13, 2015)

FEATURES:

 * **Key Rotation Support**: The `rotate` command can be used to rotate the
   master encryption key used to write data to the storage (physical) backend.
   [GH-277]
 * **Rekey Support**: Rekey can be used to rotate the master key and change the
   configuration of the unseal keys (number of shares, threshold required).
   [GH-277]
 * **New secret backend: `pki`**: Enable Vault to be a certificate authority
   and generate signed TLS certificates. [GH-310]
 * **New secret backend: `cassandra`**: Generate dynamic credentials for
   Cassandra [GH-363]
 * **New storage backend: `etcd`**: store physical data in etcd [GH-259]
   [GH-297]
 * **New storage backend: `s3`**: store physical data in S3. Does not support
   HA. [GH-242]
 * **New storage backend: `MySQL`**: store physical data in MySQL. Does not
   support HA. [GH-324]
 * `transit` secret backend supports derived keys for per-transaction unique
   keys [GH-399]

IMPROVEMENTS:

 * cli/auth: Enable `cert` method [GH-380]
 * cli/auth: read input from stdin [GH-250]
 * cli/read: Ability to read a single field from a secret [GH-257]
 * cli/write: Adding a force flag when no input required
 * core: allow time duration format in place of seconds for some inputs
 * core: audit log provides more useful information [GH-360]
 * core: graceful shutdown for faster HA failover
 * core: **change policy format** to use explicit globbing [GH-400] Any
   existing policy in Vault is automatically upgraded to avoid issues.  All
   policy files must be updated for future writes. Adding the explicit glob
   character `*` to the path specification is all that is required.
 * core: policy merging to give deny highest precedence [GH-400]
 * credential/app-id: Protect against timing attack on app-id
 * credential/cert: Record the common name in the metadata [GH-342]
 * credential/ldap: Allow TLS verification to be disabled [GH-372]
 * credential/ldap: More flexible names allowed [GH-245] [GH-379] [GH-367]
 * credential/userpass: Protect against timing attack on password
 * credential/userpass: Use bcrypt for password matching
 * http: response codes improved to reflect error [GH-366]
 * http: the `sys/health` endpoint supports `?standbyok` to return 200 on
   standby [GH-389]
 * secret/app-id: Support deleting AppID and UserIDs [GH-200]
 * secret/consul: Fine grained lease control [GH-261]
 * secret/transit: Decouple raw key from key management endpoint [GH-355]
 * secret/transit: Upsert named key when encrypt is used [GH-355]
 * storage/zk: Support for HA configuration [GH-252]
 * storage/zk: Changing node representation. **Backwards incompatible**.
   [GH-416]

BUG FIXES:

 * audit/file: file removing TLS connection state
 * audit/syslog: fix removing TLS connection state
 * command/*: commands accepting `k=v` allow blank values
 * core: Allow building on FreeBSD [GH-365]
 * core: Fixed various panics when audit logging enabled
 * core: Lease renewal does not create redundant lease
 * core: fixed leases with negative duration [GH-354]
 * core: token renewal does not create child token
 * core: fixing panic when lease increment is null [GH-408]
 * credential/app-id: Salt the paths in storage backend to avoid information
   leak
 * credential/cert: Fixing client certificate not being requested
 * credential/cert: Fixing panic when no certificate match found [GH-361]
 * http: Accept PUT as POST for sys/auth
 * http: Accept PUT as POST for sys/mounts [GH-349]
 * http: Return 503 when sealed [GH-225]
 * secret/postgres: Username length is capped to exceeding limit
 * server: Do not panic if backend not configured [GH-222]
 * server: Explicitly check value of tls_diable [GH-201]
 * storage/zk: Fixed issues with version conflicts [GH-190]

MISC:

 * cli/path-help: renamed from `help` to avoid confusion

## 0.1.2 (May 11, 2015)

FEATURES:

  * **New physical backend: `zookeeper`**: store physical data in Zookeeper.
    HA not supported yet.
  * **New credential backend: `ldap`**: authenticate using LDAP credentials.

IMPROVEMENTS:

  * core: Auth backends can store internal data about auth creds
  * audit: display name for auth is shown in logs [GH-176]
  * command/*: `-insecure` has been renamed to `-tls-skip-verify` [GH-130]
  * command/*: `VAULT_TOKEN` overrides local stored auth [GH-162]
  * command/server: environment variables are copy-pastable
  * credential/app-id: hash of app and user ID are in metadata [GH-176]
  * http: HTTP API accepts `X-Vault-Token` as auth header [GH-124]
  * logical/*: Generate help output even if no synopsis specified

BUG FIXES:

  * core: login endpoints should never return secrets
  * core: Internal data should never be returned from core endpoints
  * core: defer barrier initialization to as late as possible to avoid error
    cases during init that corrupt data (no data loss)
  * core: guard against invalid init config earlier
  * audit/file: create file if it doesn't exist [GH-148]
  * command/*: ignore directories when traversing CA paths [GH-181]
  * credential/*: all policy mapping keys are case insensitive [GH-163]
  * physical/consul: Fixing path for locking so HA works in every case

## 0.1.1 (May 2, 2015)

SECURITY CHANGES:

  * physical/file: create the storge with 0600 permissions [GH-102]
  * token/disk: write the token to disk with 0600 perms

IMPROVEMENTS:

  * core: Very verbose error if mlock fails [GH-59]
  * command/*: On error with TLS oversized record, show more human-friendly
    error message. [GH-123]
  * command/read: `lease_renewable` is now outputted along with the secret to
    show whether it is renewable or not
  * command/server: Add configuration option to disable mlock
  * command/server: Disable mlock for dev mode so it works on more systems

BUG FIXES:

  * core: if token helper isn't absolute, prepend with path to Vault
    executable, not "vault" (which requires PATH) [GH-60]
  * core: Any "mapping" routes allow hyphens in keys [GH-119]
  * core: Validate `advertise_addr` is a valid URL with scheme [GH-106]
  * command/auth: Using an invalid token won't crash [GH-75]
  * credential/app-id: app and user IDs can have hyphens in keys [GH-119]
  * helper/password: import proper DLL for Windows to ask password [GH-83]

## 0.1.0 (April 28, 2015)

  * Initial release

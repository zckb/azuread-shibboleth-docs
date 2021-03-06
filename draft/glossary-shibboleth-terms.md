# Glossary of Shibboleth Terms

**Attribute**: a piece of information about a user. Each attribute has a unique ID and has zero of more values. Shibboleth IdP attributes are protocol-agnostic data structures.

**Attribute definition**: a plugin that creates a single attribute by transforming other attributes and state information. Shibboleth currently supports simple, scoping, regex, mapping, template, scripting, principal name, and principal authentication method attribute definitions.

**Attribute encoder**: a plugin that converts an attribute into a protocol specific form, like a SAML attribute. Attribute encoders are associated with an attribute through the attribute’s attribute definition.

**Attribute filter policy**: a policy containing a trigger, that indicates if the policy is active, and a set of attribute value filters.

**Attribute resolver**: a subsystem in Shibboleth IdP responsible for fetching, transforming, and associating encoders with attributes. Only attributes produced by attribute definitions leave the resolver and are available to other parts of the Shibboleth system.

**Attribute rule**: a rule, specific to an attribute that determines which values are released to a relying party. An attribute filter policy may have any number of attribute rules.

**Authentication mechanism**: a concrete mechanism used to authenticate a user. Shibboleth IdP currently supports REMOTE_USER, username and password against LDAP and Kerberos protocol, and IP address based mechanisms.

**Authentication method**: an identifier that a relying party may use to stipulate how authentication should be performed. Authentication method identifiers correspond to a prescription of how authentication is done.

**Binding**: a description of how a SAML 2.0 message is attached to an underlying transport protocol, such as HTTP or SMTP. For example: If the message is sent over HTTP what HTTP headers need to be set, what are the URL or form parameter names, etc.

**Data connector**: a plugin that creates multiple attributes from information in data sources like Active Directory, other LDAP directory, SQL databases, etc. Shibboleth IdP supports static, LDAP, relational database, computed, and stored ID data connectors.

**Entity ID**: A unique identifier for an identity provider (IdP) or service provider (SP). In Shibboleth IdP, the recommended format is a URL IdP: https://<FQDN hostname>/idp/shibboleth, respectively SP: http://<FQDN hostname>/shibboleth.

**Login handler**: a Shibboleth IdP component that correlates all supported authentication methods with currently configured authentication mechanisms. A login handler may map more than one authentication method to the same authentication mechanism.

**Metadata**: a description of the SAML 2.0 features supported by a SAML entity. Most importantly, this includes the URLs for communicating with an entity such as Azure AD. Shibboleth IdP and SP use this information to build technical trust between entities.

**Permit value rule**: A rule that determines if an attribute value is permitted to be released to a relying party.

**Principal connector**: a plugin that converts a name identifier, provided by a relying party, into the internally used userid.

**Policy requirement rule**: a specific requirement that must be met in order for an attribute filter policy to be in effect. An attribute filter policy may only have one requirement rule but some rules allow child rules to be declared and combined.

**Profile**: a description of how to use SAML 2.0, over a specific binding, to accomplish a specific task (e.g. web Browser Single Sign-On) in an interoperable manner. Profiles are the finest grained unit of interoperability within SAML 2.0.

**Relying party**: the SAML 2.0 peer to which the IdP is communicating. In all existing cases, the relying party of the IdP is always an SP. Some very advanced cases allow one IdP to be a relying party to another IdP.

**SAML**: Security Assertion Markup Language, an XML standard for exchange authentication and authorization data between security domains.  SAML is standard set by the OASIS Security Services Technical Committee.

**SAML 2.0**: was ratified as an OASIS Standard in March 2005. Critical aspects of SAML 2.0 are covered in detail in the official documents SAML Conform, DAMLCore, SAMLBind, and SAMLProf.

**SAML attribute**: an attribute that is represented in SAML 2.0 notation. Shibboleth IdP transforms attributes into SAML attributes by a process known as encoding.

**Session state**: information about the user, currently active authentication methods, and services to which they are signed into such as Azure AD. A user’s IdP session is created the first time they authenticate but may outlive the lifetime of all authentication methods.

**Shibboleth**: is a suite of standards based, open source software packages for web single sign-on across or within organizational boundaries. It allows sites to make informed authorization decisions for individual access of protected online resources in a privacy-preserving manner.

The Shibboleth software suite implements widely used federated identity standards, principally OASIS SAML, to provide a federated single sign-on (SSO) and attribute exchange framework. Shibboleth IdP also provides extended privacy functionality allowing the browser user and their home site to control the attributes released to each application. Using Shibboleth-enabled access simplifies management of identity and permissions for organizations supporting users and applications. Shibboleth software is developed in an open and participatory environment, is freely available, and is released under the Apache Software License.

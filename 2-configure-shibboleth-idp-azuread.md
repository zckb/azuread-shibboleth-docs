## Configure Shibboleth Identity Provider for use with Azure AD Single Sign-on
This section contains guidelines on how to configure Shibboleth Identity Provider (IdP) software to be used with Azure AD to enable single sign-on access to one or more Microsoft cloud services (such as Office 365 or Microsoft Azure) using the SAML 2.0 protocol. The SAML 2.0 relying party for a Microsoft cloud service used in this scenario is Azure AD. 

This document describes what files are to be edited to appropriately configure the Shibboleth IdP. The paths used reflect our test installation and should potentially be changed to reflect your configuration.

In the rest of this document topic, the environment variable IDP_HOME is the base directory where Shibboleth IdP was installed, e.g. C:\Program Files (x86)\Shibboleth\IdP. Be sure to replace IDP_HOME with your own specific path.


## Add Azure AD metadata
Adding a partner like Azure AD into Shibboleth IdP consists in defining it in the %IDP_Home%\conf\metadata-provider.xml file. Generally speaking, this file defines how the Shibboleth IdP should interact with service providers in the federation and how it gets the federation metadata via the definition of a metadata provider.

The partner definition simply consists of referencing the partner’s XML metadata document via a new metadata provider (<MetadataProvider> element).
  
Shibboleth IdP needs information about the Azure AD relying party. Azure AD publishes metadata at https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml.

The following two metadata provider definitions enable to add the above metadata to the Shibboleth IdP:
1.	The file system metadata provider: Manually download and store Azure AD metadata in a file in the IDP_HOME/metadata folder.
-or-
2.	The file backed HTTP metadata provider: configure Shibboleth IdP to pull the Azure AD metadata directly.

It is recommended that you always import the latest Azure AD metadata when configuring your SAML 2.0 identity provider.

>[!NOTE]
>Each type of metadata provider has its own set of configuration options. For information on the metadata provider, see the online help topic [METADATACONFIGURATION](https://wiki.shibboleth.net/confluence/display/IDP30/MetadataConfiguration) on the Shibboleth Community wiki site.
>Azure AD does not read metadata from the identity provider.

1.	You also have to specify the Shibboleth IdP where to find the Azure AD metadata document. As previously discussed, you can do this by adding another entry to the metadata-provider.xml file. Still in Notepad with the metadata-provider.xml file opened, Press Ctrl+F to find “</MetadataProvider>”.
2.	Move to the next line down and insert the following text to use the file backed HTTP metadata provider:

```<!—- Microsoft Azure AD Metadata -->
<MetadataProvider id="AAD" xsi:type="FileBackedHTTPMetadataProvider"
metadataURL="https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml" 
backingFile="C:\Program Files (x86)\Shibboleth\IdP\metadata\AAD-FederationMetadata.xml" />
```

The file backed HTTP metadata provider loads the Azure AD metadata XML file via HTTP and backs it up to a local file, for example in our case, %IDP_HOME\metadata\AAD-FederationMetadata.xml.

Depending on your network configuration, you may require to interact with an HTTP proxy. In such case, additional parameters are required in the MetadataProvider element such as proxyHost and proxyPort. Please refer to the Shibboleth documentation for additional information.

If you rather want to use the file system metadata provider, the related declaration is as follows:
```
<!—- Microsoft Azure AD Metadata -->
<MetadataProvider id="AAD" xsi:type="FilesystemMetadataProvider" 
metadataFile=”C:\Program Files (x86)\Shibboleth\IdP\metadata\AAD-FederationMetadata.xml"/>
```
3.	Save and close the **metadata-provider.xml** file.

## Add Azure AD as a relying party
You must enable communication between your SAML 2.0 identity provider and Azure AD. This configuration will be dependent on your specific identity provider and you should refer to documentation for it. You would typically set the relying party ID to the same as the entityID from the Azure AD metadata.

>[!NOTE]
>Verify the clock on your SAML 2.0 identity provider server is synchronized to an accurate time source. An inaccurate clock time can cause federated logins to fail.

### Configure a relying party override 
Azure AD does not process authentication responses that are encryped, which the Shibboleth IdP does by default. 

To configure Azure AD relying party override, proceed with the following steps:
1.	Use Windows Explorer to navigate to %IDP_Home%\conf, e.g. C:\Program Files (x86)\Shibboleth\IdP\conf.
2.	Right-click the **relying-party.xml** file, and then click **Edit**. The file should open in Notepad.
3.	Press Ctrl+F to find “shibboleth.RelyingPartyOverrides”.
4.	Move to the next line down and insert the following text.

```
   <util:list id="shibboleth.RelyingPartyOverrides">
        <!—- Microsoft Azure AD -->
        <bean parent="RelyingPartyByName" c:relyingPartyIds="urn:federation:MicrosoftOnline">
            <property name="profileConfigurations">
                <list>
                    <bean parent="SAML2.SSO" p:encryptAssertions="false" />
                </list>
            </property>
        </bean>
```

Make sure that:
+The relying party id value matches the *entityID* value of the *EntityDescriptor* element of the Azure AD metadata, for example as of today *"urn:federation:MicrosoftOnline"*;

### Configuring the Shibboleth IdP attribute resolver
Azure AD requires two pieces of data from Shibboleth IdP to locate the shadow account in the authentication platform.
1.	**Azure AD ImmutableID**. Azure AD requires that you select a unique identifier for each user in your user directory. You must also configure Shibboleth IdP to send this attribute on each federated login to Azure AD in the SAML 2.0 *NameID* assertion. This identifier must not change for this user over the lifetime of the user being in your system. Azure AD Service calls this attribute the *ImmutableID*. The value for the unique identifier must not contain domain information and is case-sensitive. For example, do not use user@shibb.contoso.edu.

The value used here with AD LDS will be (as an illustration) the *uid* property that we’ve previously provisioned for the test user user1 (see section [§ CREATING AND CONFIGURE A TEST USER FOR THE SHIBBOLETH IDP]()). It can also be for example the Shibboleth eduPersonTargetedID attribute that is calculated by default.

When creating accounts, you must ensure the ImmutableID is processed the same way, otherwise the user will not be able to login to the Microsoft Cloud services. For instance, with Active Directory, the Azure AD Connect tool automatically uses the Active Directory objectGUID for the ImmutableID value and processes the ImmutableID the same way. We mimic here the approach.

2.	**Azure AD UserID**. Azure AD requires that the Azure AD User ID, for example, user@shib.idmgt.archims.fr, is sent. With Active Directory, the value is stored in the LDAP UserPrincipalName attribute. With AD LDS, we will use instead the mail attribute that we’ve provisioned earlier with the user e-mail address.

As outlined before, the Shibboleth IdP can retrieve attributes from Active Directory, another LDAP directory, a SQL database, etc., generate attributes based on other attributes, or define them statically.

For that purpose, the %IDP_Home%\conf\attribute-resolver.xml file defines how the IdP generates SAML 2.0 attributes for the IdP's users. It specifies how to configure the IdP to authenticate users against the organization's attributes source(s), e.g. AD LDS in our configuration, how to use it to look up values associated with those users, and how to use these as the basis for attribute generation.

The file more particularly defines:
1.	Data connectors (<resolver:DataConnector> element) for connecting to the attribute sources,
2.	And attribute definitions (<resolver:AttributeDefinition> element) that define the attribute type (xsi:type) and how it maps to the source (sourceAttributeID attribute).
Attribute definitions are associated with a data connector via the ref parameter of the *resolver:Dependency* child node.

>[!NOTE] 
>For information, see the online help topic [ATTRIBUTERESOLVERCONFIGURATION](https://wiki.shibboleth.net/confluence/display/IDP30/AttributeResolverConfiguration) on the Shibboleth Community wiki site. 

The Shibboleth IdP software is preconfigured to include a number of assertion attributes in the SAML 2.0 assertions it generates, including an example of *eduPersonScopedAffiliation* and *eduPersonTargetedID*. Here, we will modify the default configuration in the *attribute-resolver.xml* file to add the two above *ImmutableID* and *UserID* attributes as well the data connector for our LDAP AD LDS directory instance *ShibbolethDir*.

To inform Shibboleth of these requirements and configure the above claims type, proceed with the following steps:
1.	Use Windows Explorer to navigate to %IDP_Home%\conf, e.g. C:\Program Files (x86)\Shibboleth\IdP\conf.
2.	Right-click the **attribute-resolver.xml** file, and then click **Edit**. The file should open in Notepad.
3.	Scroll until you see the following section:

```
<!-- ========================================== -->
<!--      Attribute Definitions                 -->
<!-- ========================================== -->
```

4.	Move one line down and insert the following text:

```
    <!-- Use AD LDS objectGUID for ImmutableID -->
    <AttributeDefinition id="ImmutableID" xsi:type="Simple" sourceAttributeID="userPrincipalName">
       <Dependency ref="myLDAP" />
    </AttributeDefinition> 

    <!-- mail for Azure AD User ID -->
    <AttributeDefinition id="UserId" xsi:type="Simple" sourceAttributeID="mail">
       <Dependency ref="myLDAP" />
       <AttributeEncoder xsi:type="SAML2String" name="IDPEmail" friendlyName="UserId" />
    </AttributeDefinition>
```

5.	Move to the bottom of the file to the line before "</AttributeResolver>”, and insert the following text (copied from *attribute-resolve-ldap.xml* and the “<LDAPProperty>” element added):

```
    <DataConnector id="myLDAP" xsi:type="LDAPDirectory"         ldapURL="%{idp.attribute.resolver.LDAP.ldapURL}"         baseDN="%{idp.attribute.resolver.LDAP.baseDN}"          principal="%{idp.attribute.resolver.LDAP.bindDN}"         principalCredential="%{idp.attribute.resolver.LDAP.bindDNCredential}"         useStartTLS="%{idp.attribute.resolver.LDAP.useStartTLS:true}"         connectTimeout="%{idp.attribute.resolver.LDAP.connectTimeout}"         trustFile="%{idp.attribute.resolver.LDAP.trustCertificates}"         responseTimeout="%{idp.attribute.resolver.LDAP.responseTimeout}">         <FilterTemplate>             <![CDATA[                 %{idp.attribute.resolver.LDAP.searchFilter}             ]]>         </FilterTemplate>
	<LDAPProperty name="java.naming.ldap.attributes.binary" value="objectGUID" /> 	<ConnectionPool             minPoolSize="%{idp.pool.LDAP.minSize:3}"             maxPoolSize="%{idp.pool.LDAP.maxSize:10}"             blockWaitTime="%{idp.pool.LDAP.blockWaitTime:PT3S}"             validatePeriodically="%{idp.pool.LDAP.validatePeriodically:true}"             validateTimerPeriod="%{idp.pool.LDAP.validatePeriod:PT5M}"             expirationTime="%{idp.pool.LDAP.idleTime:PT10M}"             failFastInitialize="%{idp.pool.LDAP.failFastInitialize:false}" />     </DataConnector>
```

6.	Save and close the **attribute-resolver.xml** file.


### Configure the Shibboleth attribute filter
Shibboleth IdP must be configured to release the two previous required attributes to Azure AD.
The %IDP_HOME%\conf\attribute-filter.xml file is used to determine which attributes to release to specific service providers.
The file contains a set of attribute filter policy (*<AttributeFilterPolicy>* element) nodes that define rules (*<PolicyRequirementRule>* element) for allowing a service provider like Azure AD access to the attributes, and attribute filters that define which attributes are released.
It contains a rule which releases the transient ID to all SPs; this rule should be kept in place when you edit the attribute-filter.xml file to add your own rules.

>[!NOTE]
>For information, see the online help topic [ATTRIBUTEFILTERCONFIGURATION](https://wiki.shibboleth.net/confluence/display/IDP30/AttributeFilterConfiguration) on the Shibboleth Community wiki site.

To release the attributes to Azure AD, proceed with the following steps:
1.	Use Windows Explorer to navigate to %IDP_Home%\conf, e.g. C:\Program Files (x86)\Shibboleth\IdP\conf.
2.	Right-click the **attribute-filter.xml** file, and then click **Edit**. The file should open in Notepad.
3.	Press Ctrl+F to find “urn:mace:shibboleth:2.0:afp http://shibboleth.net/schema/idp/shibboleth-afp.xsd”.
4.	Move one line down and insert the following text to modify the list of attributes that will be released:

```
    <AttributeFilterPolicy id="PolicyForWindowsAzureAD">
       <PolicyRequirementRule xsi:type="Requester" value="urn:federation:MicrosoftOnline" />

       <!-- Release mail as Azure AD User ID -->
       <AttributeRule attributeID="UserId">
         <PermitValueRule xsi:type="ANY" />
       </AttributeRule>

       <!-- Release Immutable ID to Azure AD -->
       <AttributeRule attributeID="ImmutableID">
         <PermitValueRule xsi:type="ANY" />
       </AttributeRule>
   
    </AttributeFilterPolicy>
```

The settings showed above release the UserId and ImmutableID required attributes only to Azure AD. The settings use specific AttributeFilterPolicy IDs to indicate the attributes are required by Azure AD.
5.	Save and close the **attribute-filter.xml** file.

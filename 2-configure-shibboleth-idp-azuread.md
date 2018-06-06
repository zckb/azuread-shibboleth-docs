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

Configure a relying party override
Azure AD does not process authentication responses that are encryped, which the Shibboleth IdP does by default. 

To configure Azure AD relying party override, proceed with the following steps:
1.	Use Windows Explorer to navigate to %IDP_Home%\conf, e.g. C:\Program Files (x86)\Shibboleth\IdP\conf.
2.	Right-click the **relying-party.xml** file, and then click **Edit**. The file should open in Notepad.
3.	Press Ctrl+F to find “shibboleth.RelyingPartyOverrides”.
4.	Move to the next line down and insert the following text.



Make sure that:
+The relying party id value matches the *entityID* value of the *EntityDescriptor* element of the Azure AD metadata, for example as of today *"urn:federation:MicrosoftOnline"*;


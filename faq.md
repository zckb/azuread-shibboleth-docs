# FAQ
This is where we can update answers to Frequently Asked Questions. 

- What Azure AD-protected resources (e.g. Office 365, Azure, apps, devices) do not work with SAML 2.0 identity providers?
- Is ECP required for Exchange Online and Outlook email rich clients?
- What do I need to do to enable Modern Authentication?
- Can one Azure AD Connect server sync to two or more Azure AD directories? Last updated: June 2018
  - No. There's a 1:1 relationship between an Azure AD Connect sync server and an Azure AD tenant. For each Azure AD tenant, you need one Azure AD Connect sync server installation. For additional information, please see [Topologies for Azure AD Connect](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect-topologies). 
- Can one Azure AD Connect server sync from two or more source directories? 
  - Some scenarios are possible with Azure AD Connect, and others may require Microsoft Identity Manager (MIM) 2016. For additional information, please see [Topologies for Azure AD Connect](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect-topologies). 


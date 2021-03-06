<a name="AccessControlWithLocalSTS"></a>
#Access Control Service with local STS#

---

<a name="Overview"></a>
## Overview ##

Connecting a cloud service to its users is one of the most basic requirements of any solution, whether deployed on-premises, in the cloud or on both.

The emergence of standards is helping to break the silos which traditionally isolate accounts stored by different Websites and business entities, however offering cloud service access to users coming from multiple sources can still be a daunting task. As of today, if you want to open your cloud service to users coming from Facebook, Live ID, Google and business directories the brute-force approach demands you to lean and implement four different authentication protocols. Changes in today's world happen fast and often, forcing you to keep updating your protocol implementations to chase the latest evolutions of the authentication mechanisms of the user repositories. All this can require a disproportionate amount of energy, leaving you with fewer resources to focus on your business.

 ![A functional view of the Access Control Service](./images/A-functional-view-of-the-Access-Control-Service.png?raw=true "A functional view of the Access Control Service")

_A functional view of the Access Control Service_

Enter the Microsoft Azure Access Control Service (ACS). ACS offers you a way to outsource authentication and decouple your cloud service from all the complexity of maintaining a direct relationship with all the identity providers you want to tap from. ACS takes care of engaging every identity provider with its own authentication protocol, normalizing the authentication results in a protocol supported by the .NET framework tooling (namely the Windows Identity Foundation technology, or WIF) regardless of from where the user is coming from. WIF allows you in just few clicks to elect the ACS as the authentication manager for your cloud service; from that moment on ACS takes care of everything, including providing a UI for the user to choose among all the recognized identity providers.

Furthermore, ACS offers you greater control over which user attributes should be assigned for every authentication event; again in synergy with WIF, those attributes (called claims) can be easily accessed for taking authorization decisions without forcing the developer do understand or even be aware of the lower level mechanisms that the authentication protocols entail.

In this intermediate hands-on lab you will learn how to use the Access Control Service for managing trust relationships with multiple business identity providers. Users from two different organizations will be able to gain authenticated access to your cloud service; however you will not be required to write any special code for handling the differences between the two. You will learn how to use ACS for establishing relationships and normalizing attributes without having to touch your cloud service's source code. The lab will demonstrate how to configure ACS both via the Microsoft Azure Portal and the management API.

<a name="Objectives"></a>
### Objectives ###

In this Hands-On Lab, you will learn how to:

- Use the portal to add business identity providers through their metadata documents

- Use the portal to establish claims transformation rules for normalizing the user's attributes

- Do all of the above via management API

- Outsource authentication of a cloud service to ACS

- Use ACS to handle the home realm discovery problem

<a name="Prerequisites"></a>
### Prerequisites ###

The following is required to complete this hands-on lab:

- IIS 7 (with ASP.NET)

- [Microsoft .NET Framework 4.0][1]
- [Microsoft Visual Studio 2010][2]
- [Microsoft Windows Identity Foundation Runtime][3]
- [Microsoft Windows Identity Foundation SDK for .NET 4.0][4]
- Microsoft Windows PowerShell
- A Microsoft Azure subscription - [sign up for a free trial](http://aka.ms/WATK-FreeTrial)

[1]:http://go.microsoft.com/fwlink/?linkid=186916
[2]:http://msdn.microsoft.com/vstudio/products/
[3]:http://support.microsoft.com/kb/974405
[4]:http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504

>**Note:** This lab was designed to use Windows 7 Operating System. 

---

<a name="Setup"></a>
### Setup ###

In order to execute the exercises in this hands-on lab you need to set up your environment.

1. Open a Windows Explorer window and browse to the lab’s **source** folder.

1. Right-click the **Setup.cmd** and select **Run as administrator** to launch the setup process that will configure your environment and install the Visual Studio code snippets for this lab. This will also install the localhost certificate used by the local STS.

1. If the User Account Control dialog is shown, confirm the action to proceed.

>**Note:** Make sure you have checked all the dependencies for this lab before running the setup.

>The procedure installs a self-signed certificate to enable the SSL binding. If you already have a certificate with subject name CN=localhost, you must back up the certificate and restore after completing the steps in this guide.

<a name="CodeSnippets" /> 
### Using the Code Snippets ###

Throughout the lab document, you will be instructed to insert code blocks. For your convenience, most of that code is provided as Visual Studio Code Snippets, which you can use from within Visual Studio 2010 to avoid having to add it manually.

If you are not familiar with the Visual Studio Code Snippets, and want to learn how to use them, you can refer to the **Setup.docx** document in the **Assets** folder of the training kit, which contains a section describing how to use them.

---

<a name="Exercises"></a>
## Exercises ##

This Hands-On Lab contains one single exercise:

1. [Use Access Control Service to Federate with Multiple Business Identity Providers](#Exercise1)

> **Note:** Each exercise is accompanied by a starting solution located in the Begin folder of the exercise that allows you to follow each exercise independently of the others. Please be aware that the code snippets that are added during an exercise are missing from these starting solutions and that they will not necessarily work until you complete the exercise. Inside the source code for an exercise, you will also find an End folder containing a Visual Studio solution with the code that result from completing the steps in the corresponding exercise. You can use these solutions as guidance if you need additional help as you work through this hands-on lab.

Estimated time to complete this lab: **30 minutes** 

<a name="GettingStarted"></a>
### Getting Started: Creating a Service Namespace ###

To follow this lab and complete all the exercises you first need to create a Microsoft Azure Service Namespace. Once completed, it can be used for all of the Access Control labs and for your own projects as well.

<a name="GettingStartedTask1"></a>
#### Task 1 - Creating your Service Namespace ####

1. Navigate to [https://manage.windowsazure.com/](https://manage.windowsazure.com/). You will be prompted for your Microsoft account credentials if you are not already signed in. 

	![Microsoft Azure Login Page](images/windows-azure-login-page.png?raw=true "Microsoft Azure Login Page")

	_Microsoft Azure Login Page_

1. To use Access Control, you need to access the previous management portal version. In order to do this, hover the mouse pointer over **Preview** in the main page header and click **Take me to the previous portal**.

	![Switching to the previous portal](images/switch-to-previous-portal.png?raw=true "Switching to the previous portal")

	_Switching to the previous portal_

1. Once in the previous portal, select **Service Bus, Access Control & Caching**, located under the navigation pane.

1. Select the **Access Control** item on the Navigation pane.

1. Now you will add a **new Access Control Service Namespace**. An Access Control Service Namespace is the unique component of the addresses at which all your endpoints on the Access Control Service will be available. To do this, click **New** on the top left corner.

 	![Add Namespace](./images/Add-Namespace.png?raw=true "Add Namespace")
 
	_Add Namespace_

1. In the **Create a new Service Namespace** dialog, type in a name for your **Namespace**, select a **region**, choose a **Subscription** and click the **Create Namespace** button. Make sure to validate the availability of the name first. Service names must be **globally unique** as they are in the cloud and accessible by whomever you decide to grant access.

 	![Creating New Access Control Service Namespace](./images/Creating-New-Access-Control-Service-Namespace.png?raw=true "Creating New Access Control Service Namespace")
 
	_Creating New Access Control Service Namespace_

	Please be patient while your service is activated. It can take a few minutes while all the necessary resources are provisioned.

 	![Active Service Namespace](./images/Active-Service-Namespace.png?raw=true "Active Service Namespace")
 
 	_Active Service Namespace_
 
<a name="Exercise1"></a>
### Exercise 1: Use ACS to Federate with Multiple Business Identity Providers ###

In this exercise you are going to outsource to ACS the authentication part of a newly created web site. You will configure ACS to delegate authentication to two different business identity providers, using both the portal and the management API. If you already went through the introductory hands-on lab, you will discover that the steps you need to follow are consistent with what you had to do for using web identity providers.

In a real-life solution, the business identity providers would expose their authentication functions using Active Directory Federation Services 2 or similar packaged software offering an STS. In order to keep the system requirements simple for the lab, here you will be using a utility which runs on the local machine and simulates a proper identity provider. The steps you need to configure ACS are, however, absolutely the same as if you'd be using a real system.

> **Note:** You require a Microsoft Azure Service Namespace to complete this exercise. If you have not already done so, complete the section Getting Started: Creating a Service Namespace.

<a name="Ex1Task1"></a>
#### Task 1 - Creating the Initial Solution ####

1. Open Microsoft Visual Studio 2010 with administrator privileges. From **Start | All Programs | Microsoft Visual Studio 2010**, right-click **Microsoft Visual Studio 2010** and select **Run as administrator**.

1. Open the **WebSiteAdvancedACS.sln** empty solution file located inside the **\Source\Ex01-ACSLabsV2Federation\Begin** folder of this Lab.

1. Create a new empty website. From **File | Add | New Web Site**, select **Visual C#** in the **Installed Templates** section and then click **ASP.NET Web Site**. Change the **Web location** field to use **HTTP** and set the value to **https://localhost/WebSiteAdvancedACS**.

 	![Add New Web Site](./images/Add-New-Web-Site.png?raw=true "Add New Web Site")
 
	_Add New Web Site_

1. In the **Solution Explorer** delete the following folders from the web site:
	
	-**Account**
	
	-**Scripts**

1. And delete the following files:

	-**About.aspx**

	-**Global.asax**
 

1. Your Solution should look like the following one:

	![Solution Explorer](./images/Solution-Explorer.png?raw=true "Solution Explorer")

	_Solution Explorer_

1. Open the **Site.master** file and remove the **DIV** element with class named **"loginDisplay"** and the **NavigationMenu** menu control.

	<!-- strike:9-19,21-26 -->
	````HTML
	...
    <div class="page">
        <div class="header">
            <div class="title">
                <h1>
                    My ASP.NET Application
                </h1>
            </div>
            <div class="loginDisplay">
                <asp:LoginView ID="HeadLoginView" runat="server" EnableViewState="false">
                    <AnonymousTemplate>
                        [ <a href="~/Account/Login.aspx" ID="HeadLoginStatus" runat="server">Log In</a> ]
                    </AnonymousTemplate>
                    <LoggedInTemplate>
                        Welcome <span class="bold"><asp:LoginName ID="HeadLoginName" runat="server" /></span>!
                        [ <asp:LoginStatus ID="HeadLoginStatus" runat="server" LogoutAction="Redirect" LogoutText="Log Out" LogoutPageUrl="~/"/> ]
                    </LoggedInTemplate>
                </asp:LoginView>
            </div>
            <div class="clear hideSkiplink">
                <asp:Menu ID="NavigationMenu" runat="server" CssClass="menu" EnableViewState="false" IncludeStyleBlock="false" Orientation="Horizontal">
                    <Items>
                        <asp:MenuItem NavigateUrl="~/Default.aspx" Text="Home"/>
                        <asp:MenuItem NavigateUrl="~/About.aspx" Text="About"/>
                    </Items>
                </asp:Menu>
            </div>
        </div>
	...
	````

1. Open the **Web.config** file and remove the following sections:

	-**connectionStrings**

	-**system.web/authentication**

	-**system.web/membership**

	-**system.web/profile**

	-**system.web/roleManager**

	The **Web.config** should look like the code bellow.

	````XML
	<?xml version="1.0"?>
	<!--
	  For more information on how to configure your ASP.NET application, please visit
	  http://go.microsoft.com/fwlink/?LinkId=169433
	  -->
	
	<configuration>
	  <system.web>
	    <compilation debug="false" targetFramework="4.0" />
	  </system.web>
	
	  <system.webServer>
	     <modules runAllManagedModulesForAllRequests="true"/>
	  </system.webServer>
	</configuration>
	````

	> **Note**: All this cleanup is not strictly necessary, but it helps to keep things simple and highlight the code that will be required for integrating with ACS.

1. Press **F5** to run the Web site and ensure us that it works as expected. If an alert about **"Debugging Not Enabled"** appears, select **"Modify the Web.config file to enable debugging"** and click **OK**.

	!["Debugger Not Enabled" Alert](./images/Debugger-Not-Enabled-Alert.png?raw=true ""Debugger Not Enabled" Alert")
 
	_"Debugger Not Enabled" Alert_

	![Running the Service](./images/Running-the-Application.png?raw=true "Running the Service")
 
	_Running the Service_

1. Close the browser.

<a name="Ex1Task2"></a>
#### Task 2 - Configure one entry for the cloud service in the Access Control Service with the Microsoft Azure Portal ####

Before being able to use ACS for offloading authentication, you need to let ACS know about your cloud service. You can easily do this by working on your Microsoft Azure namespace via management portal.

1. Navigate to [https://manage.windowsazure.com/](https://manage.windowsazure.com/). You will be prompted for your Microsoft account credentials if you are not already signed in. 

1. To use Access Control, you need to access the previous management portal version. In order to do this, hover the mouse pointer over **Preview** in the main page header and click **Take me to the previous portal**.

	![Switching to the previous portal](images/switch-to-previous-portal.png?raw=true "Switching to the previous portal")

	_Switching to the previous portal_

1. Once in the previous portal, go to **Service Bus, Access Control & Caching**, located under the navigation pane.

1. Select the **Access Control** item on the Navigation pane.

1. With the **Namespace** selected, click **Access Control Service** on the top toolbar. Make sure that appservices.azure.com is allowed to show popups in your browser. 

	![Click the Access Control Service - Manage button](./images/Click-the-Access-Control-Service--Manage-button.png?raw=true "Click the Access Control Service - Manage button")

	_Click Access Control Service - Manage_

1. This launches (in another browser window or tab) **the Access Control Service Management Portal**, shown in the figure below. 

	![Access Control Service Management Portal](./images/Access-Control-Service-Management-Portal.png?raw=true "Access Control Service Management Portal")
 
	_Access Control Service Management Portal_

	> **Note:** The Management Portal offers you a global view of all the settings you can change in ACS. In this task we want to add a new cloud service: in identity jargon, you use the expression "Relying Party" for referring to an cloud service.

1. Click the **Relying Party Applications** link on the navigation menu in order to register your Web site with ACS. "Relying Party" is identity speak for cloud service, the entity which consumes identities, whereas as you already guessed "Identity Provider" indicates one entity which stores identities and is capable of authenticating users.

	![Identity Providers configured](./images/Identity-Providers-configured.png?raw=true "Identity Providers configured")

	_Identity Providers configured_

1. Click the **Add** link on top of the Relying Party Applications table and fill the form with the following values:

	
	-**Name:** WebSiteAdvancedACS
	-**Mode:** Enter settings manually
	-**Realm:** https://localhost/WebSiteAdvancedACS/
	-**Return URL:** https://localhost/WebSiteAdvancedACS/Default.aspx
	-**Error URL:** leave the field empty
	-**Token format:** SAML 1.1
	-**Token encryption policy:** None
	-**Token lifetime (secs):** 600
	-**Identity providers:** leave the default (Windows Live ID)
	-**Rule groups:** Create New Rule Group
	-**Token signing:** Use service namespace certificate (standard)


	> **Note:** Those values describe everything ACS needs to know for handling authentication for your cloud service. We'll get in more details later: here we will just say that upon successful authentication ACS sends back to the cloud service a security token  (an artifact such as an XML fragment, a piece of binary or json code, anything as long as it is digitally signed) as proof that successful authentication actually took place. In order to do so, ACS needs to know the address of the cloud service to which the token should be returned to, the desired characteristics of the token it has to create, and so on. 


	In this lab we will give you the instructions for getting the scenario up and running, but we will not go in great details about the underlying security mechanisms and protocols. If you want to know more about what happens behind the scenes, please refer to the presentations and videos section of the training kit.

	![Add Relying Party Application](./images/Add-Relying-Party-Application.png?raw=true "Add Relying Party Application")
	
	_Add Relying Party Application_
	
	> **Note:**  The _Realm_ field MUST have the trailing slash or the authentication operations will fail.

1. Click **Save**.

1. Under the **Development** section of the navigation menu, click the **Application Integration** link. Here there are various URIs that come in handy when configuring your cloud service to take advantage of ACS.

1. Go to **Endpoint Reference** section and copy the value for **WS-Federation Metadata**. You will discover what that is and what it is used for right at the beginning of the next step.

	![Copying WS-Federation Metadata](./images/Copying-WS-Federation-Metadata.png?raw=true "Copying WS-Federation Metadata")
 
	_Copying WS-Federation Metadata_

<a name="Ex1Task3"></a>
#### Task 3 - Configuring a Website to Accept Tokens from Access Control Service ####

For a cloud service, outsourcing authentication ACS means forwarding every request from unauthenticated users to ACS. ACS will do something for making authentication happen (details below), and as we have seen above the result will be a security token. All those redirects are usually done according to specific protocols, which are platform and vendor-independent.

ACS can emit different token types on various protocols. For Websites, the default protocol is WS-Federation. There's no need to go in the fine details: suffice to say that WS-Federation is a protocol that is natively understood by Windows Identity Foundation (WIF), an extension to the .NET framework that allows you to easily outsource cloud service authentication to tokens sources such as the ACS itself. In particular, WIF extends Visual Studio with a wizard which can automatically configure your cloud service to outsource authentication without requiring you to write a single line of code. All it needs is the address of a machine-readable description of the token source to be used: in our case, that description is the WS-Federation Metadata address you saved at the end of Task 3. In identity parlance, a service which emits security token is called a Security Token Service (STS).

In this task you will use the WIF wizard to outsource authentication to ACS.

1. Back to Visual Studio and in the **Solution Explorer**, right-click the **https://localhost/WebSiteAdvancedACS/** project and select **Add STS reference**.

1. When the **Federation Utility** window shows up, perform the following tasks for each step in the wizard.

	1. On the **Welcome** page click **Next** to continue using the pre-populated fields.

		![Federation Utility Wizard](./images/Federation-Utility-Wizard.png?raw=true "Federation Utility Wizard")
 
		_Federation Utility Wizard_

	1. On the **Security Token Service** page select **Use an existing STS**, and paste the endpoint taken from **Task 2 - Step 10** in the **Use an existing STS** field and click **Next**. That endpoint serves the document describing the WS-Federation STS that ACS exposes in your Microsoft Azure namespace.

		![Use an Existing STS option](./images/Use-an-Existing-STS-option.png?raw=true "Use an Existing STS option")
 
		_Use an Existing STS option_

	1. On the **STS signing certificate chain validation error** page select **Disable certificate chain validation** and click **Next**.

		![Disable certificate chain validation option](./images/Disable-certificate-chain-validation-option.png?raw=true "Disable certificate chain validation option")
 
		_Disable certificate chain validation option_

	1. On the **Security token encryption** page select **No encryption** and click **Next**.

		![Security Token encryption](./images/Security-Token-encryption.png?raw=true "Security Token encryption")
 
		_Security Token encryption_

	1. On the **Offered claims** page click **Next**.

		![Offered Claims](./images/Offered-Claims.png?raw=true "Offered Claims")
 
		_Offered Claims_

		> **Note:** The WS-Federation Metadata can contain descriptions of the claims that the endpoint offers. The wizard shows those to you so that you will know what information about incoming users you will be able to process in your cloud service. In this case ACS declares that it can release information about which identity provider was actually used for authentication and a user identifier.

	1. On the **Summary** page review the changes that will be made and click **Finish**.

<a name="Ex1Task4"></a>
#### Task 4 - Use the ACS Management Portal to Trust a Business Identity Provider and Process User Attributes via Claims Mapping Rules ####

![ACS does not directly authenticate users, but it brokers authentication between your cloud service and ](./images/ACS-does-not-directly-authenticate-users,-but-it-brokers-authentication-between-your-application-and-.png?raw=true "ACS does not directly authenticate users, but it brokers authentication between your cloud service and ")
 
_ACS does not directly authenticate users, but it brokers authentication between your cloud service and multiple providers. The picture shows a simplified authentication flow, which will be described in the following tasks_

ACS does not directly authenticate users: in the most common cases it does not maintain credentials such as username and password, but it rather brokers authentication between your cloud service and other providers. Let's say that you are developing an inventory cloud service for your warehouse, and two partner companies need to have access to it in order to sell your goods. You want the employees of those two partners to have authenticated access to your cloud service, but you don't want to manage their identities. Here's where ACS comes to the rescue: assuming that those companies expose their own STSs as well, ACS will simply take care to redirect unauthenticated requests to one or the other, process the resulting token and send back a new authentication token to your cloud service. That way, users never have to disclose their credentials outside of their own infrastructure and you never have to manage credentials you don't own. Organizations which expose via STS their capability of authenticating users are called Identity Providers (IP for short).

In this task you will use the portal for configuring ACS to accept users from the first of two identity providers.

1. Back to the browser, click the **Identity Providers** link in the **Trust Relationships** section of the menu. The main area of the management portal will display a page which helps you to manage the identity providers from where your cloud service users will come from.

	![Identity Providers](./images/Identity-Providers.png?raw=true "Identity Providers")

	_Identity Providers_

1. Click the **Add** link above the Identity Providers table, choose **WS-Federation identity provider** and click **Next**.  

	![Adding Identity Provider](./images/Adding-Identity-Provider.png?raw=true "Adding Identity Provider")

	_Adding Identity Provider_

	> **Note:** ACS is able to broker authentication with many different types of identity providers. Web IPs such as Windows Live ID, Google, Yahoo and Facebook are all services available on the public internet, defined by the address of their STS (or equivalent), the set of attributes (claims) they share about their users and the authentication protocol they use.

	> Business IPs, conversely, behaves in a slightly different way. Every company will have its own STS address, will share different claims about their users, and so on. In ACS you can add multiple business IPs: one of the advantages of ACS is exactly that it can help you to manage many trust relationship without burdening your cloud service code.

	> As of today, the protocol that ACS uses for handing business IPs for cloud services is WS-Federation. On the Windows platform the standard way of exposing an STS is using Active Directory Federation Services 2.0, which is why the "Add" button for adding a business IP is labeled as in Figure 22. However any WS-Federation STS should work, even from non-Windows platform (WS-Federation is an open, vendor-independent standard).

1. Complete the form with the following information: 

	-**Identity Provider Settings**

	-Display Name: _SelfSTS1_

	-WS-Federation metadata_:_ Select **File**, and then **Browse** the file  _Source\Assets\SelfSTS1\FederationMetada.xml  in the lab's folder_

	-**Login Page Settings**

	-Login link text: _SelfSTS1_

	-Image URL:  _(leave blank)_

	-Email domain name(s): _(leave blank)_

	-**Used by**

	-Relying party applications: _WebSiteAdvancedACS_

	![Filling information about the first business IP in the ACS portal](./images/Filling-information-about-the-first-business-IP-in-the-ACS-portal.png?raw=true "Filling information about the first business IP in the ACS portal")
 
	_Filling information about the first business IP in the ACS portal_

	> **Note:** The form you just filled is the functional equivalent of the wizard you ran on your cloud service back on task 3, this time applied to ACS. The wizard configured the cloud service to redirect unauthenticated requests to the ACS and accept the resulting tokens as proof of authentication; this form is configuring ACS to redirect unauthenticated requests to the first business IP and accept the tokens it emits as authentication proofs.

	> Below you will learn more details about the STS we are using for simulating business IPs in this lab.

1. Click **Save**.

1. In the **Trust Relationships** section of the navigation menu, click the **Rule Groups** link in order to select the default rule group for your cloud service.

	![The current rule groups list contains just the default rule group ](./images/The-current-rule-groups-list-contains-just-the-default-rule-group-.png?raw=true "The current rule groups list contains just the default rule group ")
 
	_The current rule groups list contains just the default rule group_

	> **Note**: A very important aspect of security tokens is that they contain claims, attributes describing the authenticated user as asserted by the originating STS. The claims, which can be pretty much anything about the user (name, email, group memberships, privileges, spending limits and so on), provide key information which drive the authentication and authorization process.  ACS provides a rule engine which can process the claims received in the incoming token, and include the resulting transformed claims in the token sent back to the cloud service. Often the claims in the output token will be simple pass-through of the values received from the IP, but in many cases ACS will perform important transformations such as assigning cloud service-specific roles to the incoming user on the basis of, for example, their group memberships in their originating organization. In the steps below you will learn how to set up some simple transformation rules.

1. Click **Default Rule Group for** **WebsiteAdvacedACS**.

1. Click the **Add** link.

	![Add Rule link](./images/Add-Rule-link.png?raw=true "Add Rule link")

	_Add Rule link_

1. Complete the rule with the following values:

	-**If ...**

	-**Claim issuer:** Select _**Identity Provider**,_ and then select _SelfSTS1_ in the dropdown

	-**Input claim type:** Select _**Select Type**_, and then select _http://selfsts1.com/claims/name_ value in the dropdown

	-**Input claim value:** Select _**Any**_

	-**Then** ...

	-**Output claim type:** Select _**Select type**_, and then select _shttp://schemas.xmlsoap.org/ws/2005/05/identity/claims/name_

	-**Output claim value:** Select _**Pass through input claim value**_

	-**Rule information**

	-**Description:** Pass through "name" claim from SelfSTS1 as "name"

	![Adding name pass through Rule](./images/Adding-name-pass-through-Rule.png?raw=true "Adding name pass through Rule")
 
	_Adding name pass through Rule_

1. Click **Save**.

1. Repeat the previous steps to add the following 3 additional rules:

	| **Rule 2** | **Value** |
	| ---------- | --------- |
	| **Claim Issuer** | **Identity Provider** - SelfSTS1 |
	| **(And) Input claim type** | **Select Type** -  http://selfsts1.com/claims/emailaddress |
	| **(And) Input claim value** | **Any** |
	| **Output claim type** | **Select Type** - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress |
	| **Output claim value** | **Pass through input claim value** |
	| **Description** | Pass through "emailaddress" claim from SelfSTS1 as "emailaddress" |
  
	| **Rule 3** | **Value** |
	| ---------- | --------- |
	| **Claim Issuer** | **Identity Provider** - SelfSTS1 |
	| **(And) Input claim type** | **Select Type** -  http://selfsts1.com/claims/Group |
	| **(And) Input claim value** | **Enter value** -  Administrators  |
	| **Output claim type** | **Enter Type** - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/role |
	| **Output claim value** | **Enter Value** - Gold |
	| **Description** | Map Gold Rule |

	| **Rule 4** | **Value** |
	| ---------- | --------- |
	| **Claim Issuer** | **Identity Provider** - SelfSTS1 |
	| **(And) Input claim type** | **Select Type** - http://selfsts1.com/claims/Group |
	| **(And) Input claim value** | **Enter value**  - Users |
	| **Output claim type** | **Enter Type** - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/role |
	| **Output claim value** | **Enter Value** - Silver |
	| **Description** | Map Silver Rule |

	> **Note:** The first three rules you have added just pass though the name, group and email claims. Rule #3 and #4 map the group claim from SelfSTS1 the business IP to a role claim in ACS: Administrators and Users roles are map to Gold and Silver roles respectively. This is a great way of keeping your cloud service code untainted from organization-specific considerations. Your cloud service is just concerned about if the current user is silver or gold and enforces access rights accordingly. If the business aspects of the partnership changes, and from now on both Users and Administrator roles should now be awarded the Gold role, all you need to do is change rule 4: there is no need to touch the cloud service code.

1. In the **Edit Rule Group** page, verify the rules you have just created and click **Save** .

	![Saving group rules](./images/Saving-group-rules.png?raw=true "Saving group rules")
 
	_Saving group rules_

1. You have completed the entire configuration for the SelfSTS1 first business IP in ACS and setup your web site to trust ACS. Now you will verify its behavior by running the SelfSTS1. It's time to verify that everything works as expected. To do this,  execute the SelfSTS.exe file located in **\Source\Assets\SelfSTS1**.

	> **Note:**  In realistic settings, the business IP would expose its STS via ADFS2.0. However that would require quite a lot of infrastructure, including Active Directory and a Windows Server machine on which to turn on the necessary server role. However we want you to be able to experiment with this scenario even if all you have available is a standalone PC. To that purpose this hands-on lab uses SelfSTS, a simple utility which exposes a minimal WS-Federation STS endpoint. SelfSTS is just a Windows Forms application, which does not even require a setup and can run on any system where the WIF runtime is available. SelfSTS can be used as a test STS when developing Websites secured with Windows Identity Foundation. You can find more information in SelfSTS MSDN code page. 

	> All the tasks you are required to perform on ACS as application developer in order to configure a business IP are precisely the same you would do if instead of SelfSTS you would be using ADFS2.0.

	![The SelfSTS utility is here used for simulating the first business IP in the scenario](./images/The-SelfSTS-utility-is-here-used-for-simulating-the-first-business-IP-in-the-scenario.png?raw=true "The SelfSTS utility is here used for simulating the first business IP in the scenario")
 
	_The SelfSTS utility is here used for simulating the first business IP in the scenario_

1. Click **Start**:  the SelfSTS endpoint will start listening for requests on the indicated port.

	![The SelfSTS is now listening for requests on the specified local port](./images/The-SelfSTS-is-now-listening-for-requests-on-the-specified-local-port.png?raw=true "The SelfSTS is now listening for requests on the specified local port")
 
	_The SelfSTS is now listening for requests on the specified local port_

1. Back to the browser, click the **Relying Party Applications** link under the **Trust Relationships** section.

1. Click on **WebSiteAdvancedACS** Relying Party.

	![The WebSiteAdvancedACS Relying Party configured in ACS](./images/The-WebSiteAdvancedACS-Relying-Party-configured-in-ACS.png?raw=true "The WebSiteAdvancedACS Relying Party configured in ACS")

	_The WebSiteAdvancedACS Relying Party configured in ACS_

1. In the **Edit Relying Party Application** page uncheck the **Windows Live ID** option into the Identity providers list, and click **Save**.

	![Removing Windows Live ID Identity Provider](./images/Removing-Windows-Live-ID-Identity-Provider.png?raw=true "Removing Windows Live ID Identity Provider")
 
	_Removing Windows Live ID Identity Provider_

	> **Note:** Windows Live ID is the IP that ACS adds as the initial choice when you create a Relying Party. For development purposes it is a reasonable default, as it is the only IP for which it is certain that the developer has a relationship with (a Windows Live ID account is needed for signing up for ACS labs and creating a Microsoft Azure namespace). However in this scenario we are only interested in handling identities from two specific business IPs, hence we are deselecting it.

	> It is interested to notice that if we'd wish to address scenarios in which users can come both from web IPs (Windows Live ID, Facebook, Google, Yahoo) and business IPs ACS would easily handle that.

1. Back to Visual Studio, press **F5** to run the Web site.	

1. The relying party cloud service _(https://localhost/WebSiteAdvancedACS/)_ will redirect to the **Access Control Service** to authenticate.

1. Access Control sent to our cloud service the claims it was expecting and we are now authenticated.

	![User Authenticated](./images/User-Authenticated.png?raw=true "User Authenticated")
 
	_User Authenticated_

	> **Note:** If you carefully observe the address bar in your browser as it opens the cloud service, you can see the how the entire redirect sequence takes place: first the Web Site, then the STS, and back to the Web site. If you want to see the flow in more details, you can take advantage of utilities such as Fiddler or the Internet Explorer 9 traffic capture utility.

	> **Note:** In order to keep this hands-on lab simple, we didn't add any code to the Web site which would take advantage of the incoming claims (i.e. giving access to a certain page to gold users but not to others). WIF makes it very easy: if you are interested in learning how to leverage the incoming claims in your cloud service access strategy, please refer to the WIF hands-on labs in this training kit.

1. Close the browser.

<a name="Ex1Task5"></a>
#### Task 5 - Use the ACS Management API to Trust a Second Business Identity Provider and Create Claims Mapping Rules. ####

The ACS Management portal is very convenient for managing trust relationships and transformation rules. However, there are scenarios in which an interactive approach is not feasible. For example, you may need to automate the onboarding of a new IP as part of an existing batch process, or change access rules in response to programmatic events. For those cases, ACS offers an exhaustive OData based API which allows you to change all the settings you can work on with the portal, and more. The next task will give you a taste of what you can do with the management API: you will add a second business IP, simulated by another SelfSTS instance, and configure the associated claims transformation rules all via API.

1. Back to the browser, click the **Management service** link under the **Administration** section of the menu.

1. On the **Management Service Accounts** section of the **Management Service** page, click the **ManagementClient** link.

	![Selecting ManagementClient](./images/Selecting-ManagementClient.png?raw=true "Selecting ManagementClient")
 
	_Selecting ManagementClient_

1. In the **Credentials** section, click the **Password** link.

 	![Access Control Management Service Identity Password.](./images/Access-Control-Management-Service-Identity-Password..png?raw=true "Access Control Management Service Identity Password.")
 
	_Access Control Management Service Identity Password._

	> **Note:** All calls to the management API are authenticated, of course. You need to use the credentials defined in ACS for this namespace.

1. Copy the value of the **Password** field into a Notepad, as you will use it in the following steps, and then click **Cancel**.

	![Copying the management key](./images/Copying-the-management-key.png?raw=true "Copying the management key")
 
	_Copying the management key_

1. Back to Visual Studio, add the **ManagementServiceProject** project. To do this, right-click the **WebSiteAdvancedACS** solution and select **Add | Existing Project**

1. In the **Add Existing Project** dialog, select the **ManagementServiceProject.csproj** file under the **\Source\Assets\ManagementService\** folder for this lab and click **Open**.

	![Adding the ManagementServiceProject](./images/Adding-the-ManagementServiceProject.png?raw=true "Adding the ManagementServiceProject")
 
	_Adding the ManagementServiceProject_

1. In the **ManagementServiceProject**, open the **ManagementServiceHelper.cs** file.

1. Update the following values. Change the **{yourManagementServiceKey}** placeholder with the password copied in notepad. Change the **{youServiceNamespace}** placeholder with your ServiceNamespace and save the file.

	````C#
	static string serviceIdentityUsernameForManagement = "ManagementClient";
	static string serviceIdentityPasswordForManagement = **"{yourManagementServiceKey}";**
	
	static string serviceNamespace = **"{yourServiceNamespace}"**;
	static string acsHostName = "accesscontrol.windows.net";
	````

1. Add a new console application project named **IdentityProviderSetup**. To do this, right-click the **WebSiteAdvancedACS** solution and select **Add | New Project.** Then choose the Console Application template, update the name to **IdentityProviderSetup** and click **OK.**

	![Creating a new Console Application](./images/Creating-a-new-Console-Application.png?raw=true "Creating a new Console Application")
 
	_Creating a new Console Application_

1. Right-click the **IdentityProviderSetup** project and select **Properties**.

1. In the **Application** tab, update the **Target Framework** to **.Net Framework 4**. In the **Target Framework change** message dialog, click **Yes** to reload the project.

	![Updating the project to target .Net Framework 4](./images/Updating-the-project-to-target-.Net-Framework-4.png?raw=true "Updating the project to target .Net Framework 4")
 
	_Updating the project to target .Net Framework 4_

1. Create a new folder named **Resources**. To do this, right-click the **IdentityProviderSetup** project, and click **Add | New folder**. Name the new folder **Resources**.

1. Add the **SelfSTS.cer** file located in **\Source\Assets\SelfSTS2\** in the folder you just created.

	![Adding the Certificate for the second Identity Provider](./images/Adding-the-Certificate-for-the-second-Identity-Provider.png?raw=true "Adding the Certificate for the second Identity Provider")
 
	_Adding the Certificate for the second Identity Provider_

	> **Note:** As mentioned, tokens are digitally signed. In order to trust an IP, ACS needs to know which key should be used for verifying this specific signature. Here we are preparing SelfSTS.cer, the key that ACS should use for verifying tokens coming from SelfSTS, to be uploaded as part of our trust relationship creation.

1. Right click **Resources\SelfSTS.cer** and on the properties check that **Copy to Output Directory** is **Copy always**.

	![Configuring copy always on the Certificate file](./images/Configuring-copy-always-on-the-Certificate-file.png?raw=true "Configuring copy always on the Certificate file")
 
	_Configuring copy always on the Certificate file_

1. In the **IdentityProviderSetup** project add a reference to**Microsoft.IdentityModel**,**System.Data.Services.Client** assemblies and to the **ManagementServiceProject** project**.**

1. Open **Program.cs** and add the following **bolded** using statements.

	(Code Snippet - _ACSwithLocalSTS - Ex01 Task5 01 - Using Statements_ - C#)

	````C#
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;	using System.Data.Services.Client;
	using System.IO;
	using ACS.Management;
	using Common.ACS.Management;
	````

1. Add the following internal class in **Program.cs** file.

	(Code Snippet - _ACSwithLocalSTS - Ex01 Task5 02- RuleTypes Class_ - C#)

	````C#
	internal class RuleTypes
	{
	  public const string Simple = "Simple";
	  public const string Passthrough = "Passthrough";
	}
	````

1. To create a new **Identity Provider** add the following method to the **Program.cs** file inside the **Program** class.

	(Code Snippet - _ACSwithLocalSTS - Ex01 Task5 03 - CreateIdpManually Method_ - C#)

	````C#
	/// <summary>
	/// Add an Identity Provider
	/// </summary>
	private static Issuer CreateIdpManually(DateTime startDate, DateTime endDate, 
ManagementService svc0, string idpName, string idpDisplayName, string idpAddress, 
string idpKeyDisplayName)
	{
	    var issuer = new Issuer
	    {
	        Name = idpName
	    };
	
	    // Check the Issuer does not exist previouly (if it exists, delete it)            
	    var oldIssuer = svc0.Issuers.Where(ip => ip.Name == issuer.Name).FirstOrDefault();
	    if (oldIssuer != null)
	    {
	        svc0.DeleteObject(oldIssuer);
	        svc0.SaveChanges();
	    }
	
	    // Add Issuer
	    svc0.AddToIssuers(issuer);
	    svc0.SaveChanges(SaveChangesOptions.Batch);
	    Console.WriteLine("Info: Issuer created: {0}", idpName);
	
	    var idp = new IdentityProvider
	    {
	        DisplayName = idpDisplayName,
	        LoginLinkName = idpDisplayName,
	        WebSSOProtocolType = "WsFederation",
	        IssuerId = issuer.Id
	    };
	
	    // Check the IP does not exist previouly (if it exists, delete it)            
	    var oldIdentityProvider = svc0.IdentityProviders.Where(ip => ip.DisplayName ==
idp.DisplayName).FirstOrDefault();
	    if (oldIdentityProvider != null)
	    {
	        svc0.DeleteObject(oldIdentityProvider);
	        svc0.SaveChanges();
	    }
	            
	    // Add the new IP to ACS
	    svc0.AddObject("IdentityProviders", idp);
	
	    // Console.WriteLine("Info: Identity Provider created: {0}", idp.Name);
	    Console.WriteLine("Info: Identity Provider created: {0}", idp.DisplayName);
	
	    // Identity provider public key to verify the signature
	    var cert = File.ReadAllBytes(@"Resources\SelfSTS.cer");
	    var key = new IdentityProviderKey
	    {
	        IdentityProvider = idp,
	        DisplayName = idpKeyDisplayName,
	        EndDate = endDate,
	        StartDate = startDate,
	        Type = "X509Certificate",
	        Usage = "Signing",
	        Value = cert
	    };
	
	    svc0.AddRelatedObject(idp, "IdentityProviderKeys", key);
	    svc0.SaveChanges(SaveChangesOptions.Batch);
	
	    Console.WriteLine("Info: Identity Provider Key added: {0}", 
idpKeyDisplayName);
	
	    // WS-Federation sign-in URL
	    var idpaSignIn = new IdentityProviderAddress
	    {
	        IdentityProviderId = idp.Id,
	        EndpointType = "SignIn",
	        Address = idpAddress
	    };
	
	    svc0.AddRelatedObject(idp, "IdentityProviderAddresses", idpaSignIn);
	    svc0.SaveChanges(SaveChangesOptions.Batch);
	
	    Console.WriteLine("Info: Identity Provider Address added: {0}", idpAddress);
	
	    string labRelyingPartyName = "WebSiteAdvancedACS";
	
	    // Relying Party related to the Identity Provider
	    foreach (var existingRelyingParty in svc0.RelyingParties)
	    {
	        var rpid = new RelyingPartyIdentityProvider
	        {
	            IdentityProviderId = idp.Id,
	            RelyingPartyId = existingRelyingParty.Id
	        };
	        existingRelyingParty.RelyingPartyIdentityProviders.Add(rpid);
	        idp.RelyingPartyIdentityProviders.Add(rpid);
	        svc0.AddToRelyingPartyIdentityProviders(rpid);
	    }
	    svc0.SaveChanges(SaveChangesOptions.Batch);
	
	    Console.WriteLine("Info: Relying Party added to Identity Provider: {0}", 
labRelyingPartyName);
	
	    return issuer;
	}
	````

	> **Note:** This is quite a lot of code, but if you spend few moments looking at it, you'll discover that it is very straightforward: it just provides the same info you'd add via portal UI when adding a new business identity provider and does some housekeeping with the collections.

1. To create new **Rules** to an existing **Rule Group** add the following method inside the **Program** class.

	(Code Snippet - _ACSwithLocalSTS - Ex01 Task5 04 - AddRulesToRuleGroup Method_ - C#)

	````C#
	/// <summary>
	/// Add the Rules into a Rule Group.
	/// </summary>
	private static void AddRulesToRuleGroup(string ruleGroupName, string issuerName)
	{
	    ManagementService svc = ManagementServiceHelper.CreateManagementServiceClient();
	
	    RuleGroup rg = svc.RuleGroups.AddQueryOption("$filter", "Name eq '" + 
ruleGroupName + "'").FirstOrDefault();
	            
	    Issuer issuer = svc.Issuers.Where(i => i.Name == issuerName).ToArray()[0];
	
	    Rule namePassthroughRule = new Rule()
	    {
	        Issuer = issuer, IssuerId = issuer.Id,
	        
	        //InputClaimIssuerId = issuer.Id,
	        InputClaimType = 
"http://www.theselfsts2.net/claims/nome",
	        OutputClaimType = 
"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
	        RuleGroup = rg,
	        //RuleType = RuleTypes.Passthrough,                
	        Description = "Passthrough \"nome\" claim from SelfSTS2 as \"name\""
	    };
	    
	    svc.AddRelatedObject(rg, "Rules", namePassthroughRule);
	
	    Rule emailPassthroughRule = new Rule()
	    {
	        Issuer = issuer, IssuerId = issuer.Id, 
	        InputClaimType = "http://www.theselfsts2.net/claims/postaelettronica",
	        OutputClaimType = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
	        RuleGroup = rg,                
	        Description = "Passthrough \"postaelettronica\" claim from SelfSTS2 as \"emailaddress\""
	    };
	    
	    svc.AddRelatedObject(rg, "Rules", emailPassthroughRule);
	
	    Rule goldenRule = new Rule()
	    {
	        Issuer = issuer,
	        IssuerId = issuer.Id, 
	        InputClaimType = "http://www.theselfsts2.net/claims/gruppo",
	        InputClaimValue = "Amministratori",
	        OutputClaimType = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/role",
	        OutputClaimValue = "Gold",
	        RuleGroup = rg,                
	        Description = "Map Gold Role SelfSTS2"
	    };
	    
	    svc.AddRelatedObject(rg, "Rules", goldenRule);
	
	    Rule silverRule = new Rule()
	    {
	        Issuer = issuer, 
	        IssuerId = issuer.Id, 
	        InputClaimType = "http://www.theselfsts2.net/claims/gruppo",
	        InputClaimValue = "Utenti",
	        OutputClaimType = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/role",
	        OutputClaimValue = "Silver",
	        RuleGroup = rg,                
	        Description = "Map Silver Role SelfSTS2"
	    };
	    
	    svc.AddRelatedObject(rg, "Rules", silverRule);
	
	    svc.SaveChanges(SaveChangesOptions.Batch);
	
	    Console.WriteLine();
	    Console.WriteLine("-----------------------------------");
	    Console.WriteLine("Info: Passthrough Rules:");
	    Console.WriteLine("Info: Passthrough Name Rule created: {0}",
 namePassthroughRule.Description);
	    Console.WriteLine("Info: Passthrough Email Rule created: {0}",
 emailPassthroughRule.Description);
	    
	    Console.WriteLine();
	    Console.WriteLine("-----------------------------------");
	    Console.WriteLine("Info: Roles Rules:");
	    Console.WriteLine("Info: Golden Rule created: {0}", goldenRule.Description);
	    Console.WriteLine("Info: Silver Rule created: {0}", silverRule.Description);
	}
	````

	> **Note:** Once again, it seems a lot of code but in fact it is roughly proportional to the data we are providing.

	> If you observe the code above, you will notice that the claim types emitted by the second business identity provider are different: in this specific case, they are in Italian language. This is a typical case in which, without an intermediary such as ACS to decouple your cloud service from the identity providers, you'd have to write specific code in order to accommodate the differences between how different partners manage and represent their users' information.

	> Using ACS allows you to write rules that will process the incoming claims, extract the information you need and create a normalized token which has consistent format regardless of the original data it was created from. Your cloud service just needs to care about if the current user is gold or silver, and needn't worry about the difference between "Amministratori" and "Administrators".

1. Now add the following method, which uses the previously created methods to create an Identity Provider with corresponding rules.

	(Code Snippet - _ACSwithLocalSTS - Ex01 Task5 05 - CreateIdentityProviderWithRules Method_ - C#)

	````C#
	private static void CreateIdentityProviderWithRules()
	{
	            ManagementService svc = ManagementServiceHelper.CreateManagementServiceClient();
	
	            //Create Identity Provider
	            var issuer =
CreateIdpManually(DateTime.UtcNow, DateTime.UtcNow.AddYears(1), svc,
 "SelfSTS2", "SelfSTS2", "http://localhost:9000/STS/Issue/", "IdentityTKStsCertForSigning");
	
	            //Add the Rules
	            string ruleGroupname = "Default Rule Group for WebSiteAdvancedACS";
	            AddRulesToRuleGroup(ruleGroupname, issuer.Name);
	
	            Console.WriteLine("Done!");
	}
	````

	> **Note:** The ManagementService class is pretty straightforward, mainly plumbing for performing the OData calls.

1. Finally, change the Main method with the following code.

	(Code Snippet - _ACSwithLocalSTS - Ex01 Task5 06 - Update Main Method_ - C#)

	````C#
	  public static void Main(string[] args)
	  {
	       CreateIdentityProviderWithRules();
	
	       Console.ReadLine();
	  }
	````

1. Right-click the **IdentityProviderSetup** project and select **Set as StartUp Project**.

1. Press **F5** to run the console application. You can see the console and verify that the Identity Provider and the rules were created.

	![The output on the console of the identity provider and rules creation code](./images/The-output-on-the-console-of-the-identity-provider-and-rules-creation-code.png?raw=true "The output on the console of the identity provider and rules creation code")
 
	_The output on the console of the identity provider and rules creation code_

1. You can also verify that the Identity Provider and the rules were created navigating back to the portal. To do this, in the **Edit Management Service Account** page, click the **Return to Management Service** link.

1. Click **Return to Access Control Service** to go back to the **Access Control Service** page.

1. In the **Trust Relationships** section, click the **Identity Providers** link. Note that the SelfSTS2 Identity Provider was created.

	![SelfSTS2 Identity Provider verification](./images/SelfSTS2-Identity-Provider-verification.png?raw=true "SelfSTS2 Identity Provider verification")
 
	_SelfSTS2 Identity Provider verification_

1. Close the browser.

1. To verify that the new SelftSTS2 Identity Provider is working as expected, execute the **SelfSTS.exe** file located in **\Source\Assets\SelfSTS2** folder of this lab.

	![The second SelfSTS instance represents the second Identity Provider](./images/The-second-SelfSTS-instance-represents-the-second-Identity-Provider.png?raw=true "The second SelfSTS instance represents the second Identity Provider")
 
	_The second SelfSTS instance represents the second Identity Provider_

1. Click **Start**.

	![The second SelfSTS instance now listening for requests](./images/The-second-SelfSTS-instance-now-listening-for-requests.png?raw=true "The second SelfSTS instance now listening for requests")
 
	_The second SelfSTS instance now listening for requests_

	> **Note:** We now have two instances of the SelfSTS utility running, listening on different local ports. In a realistic system the IPs would likely live on completely different systems, representing distinct business entities.

1. Back to Visual Studio, right-click the https://localhost/WebSiteAdvancedACS/ project and select **Set as StartUp Project.** 

1. Press **F5** to start the Web Site.

1. Verify that the new Identity Provider **SelfSTS2** appears in the **Sign In** form (you might need to click on the **Show more options** link).

	![The ACS HDR default page](./images/The-ACS-HDR-default-page.png?raw=true "The ACS HDR default page")
 
	_The ACS HDR default page_

	> **Note:** When we had only one business IP configured in ACS, the authentication phase took place transparently if you exclude some changes in the address bar. Now we have two possible IPs to which we can redirect the user for authenticating, and at this point we have no clue which ACS could use to decide whether to redirect with one or the other.  The problem of taking this decision is known as the Home Realm Discovery problem, or HRD, and it arises almost every time there are multiple choices of identity providers. ACS offers various tools to address HDR. The one you can see in Figure 47 is the most direct, in which ACS automatically generates a page with as many buttons as there are configured IPs. Another approach consists of prompting the user for their email address, and using the domain part of it for establishing which IP is the user from (see Figure 24). The advantage of the second approach is that it does not reveal upfront the identity of the IP on the page.

	> Both approaches can be mixed and matched, and ACS can even provide the HDR information programmatically so that you can build your own chrome for the HDR, for example if you want to maintain a consistent look & feel with the rest of your cloud service.

1. Close the browser.

1. You will now use the Security Token Visualizer Control to see the different claims provided by ACS. To do this, back to Visual Studio, open the **Default.aspx** file in the **WebSiteAdvancedACS** project.  

1. From the **Visual Studio Toolbox** drag and drop a **Security Token Visualizer Control** at the bottom of the main content control into the **Default.aspx** page:

	![The Security Token Visualizer Control in Toolbox](./images/The-Security-Token-Visualizer-Control-in-Toolbox.png?raw=true "The Security Token Visualizer Control in Toolbox")
 
	_The Security Token Visualizer Control in Toolbox_

	> **Note:** The control we use here is a fast way of obtaining debug information about the incoming security token without having to write any code: however accessing claims information from your code-behind is very easy, please refer to the WIF related hands-on lab in this training kit if you want to know more.

	````HTML
	...
	<cc1:SecurityTokenVisualizerControl ID="SecurityTokenVisualizerControl1" runat="server" />
	<asp:content/>
	````

	> **Note:** In case no code appears when you drop the control on the page, please close Visual Studio, run the Setup Scripts again and restart Visual Studio.

1. Open the **Web.config** file in the https://localhost/WebSiteAdvancedACS/ project  and enable the **saveBootstrapTokens** attribute inside **microsoft.identityModel** section:

	````XML
	<microsoft.identityModel>
	  <service saveBootstrapTokens="true">
	    <audienceUris>
	````

1. Press **F5** to start debugging the cloud service.

1. In the **Sign in** page, choose **SelfSTS1** Identity Provider in order to authenticate into the Website (you might need to click on the **Show more options** link).

1. See the specific claims of the SelfSTS1 Identity Provider.

	![The Security Token Visualizer Control showing the content of the token coming from the first ](./images/The-Security-Token-Visualizer-Control-showing-the-content-of-the-token-coming-from-the-first-.png?raw=true "The Security Token Visualizer Control showing the content of the token coming from the first ")
  
	_The Security Token Visualizer Control showing the content of the token coming from the first business IP_
1. Close the browser and press **F5** to run the solution again.

1. Choose **SelfSTS2** Identity Provider in order to authenticate into the Website (you might need to click on the **Show more options** link).

1. See the specific claims of the SelfSTS2 IP.

	![The Security Token Visualizer Control showing the content of the token coming from the ](./images/The-Security-Token-Visualizer-Control-showing-the-content-of-the-token-coming-from-the-.png?raw=true "The Security Token Visualizer Control showing the content of the token coming from the ")
 
	_The Security Token Visualizer Control showing the content of the token coming from the second business IP_

1. Close the browser.

<a name="Ex1Summary"></a>
#### Exercise 1: Summary ####

In this exercise you took advantage of ACS for brokering authentication between a Web site and multiple business identity providers: that is a very common scenario, a problem you'd have to solve every time you want to open access for business partners to one cloud service of yours, be it a level of business app or a Software as a Service (SaaS) solution. You had the chance to experience how the presence of ACS in the architecture can simplify the task of establishing trust relationships and keeping your cloud service insulated from changes and differences in how the partners handle user information. ACS can be configured via management portal or via management API: in this hands-on lab you experienced with both approaches.

---

<a name="Summary"></a>
## Summary ##

By completing this Hands-On Lab you have learned how to:

- Use the portal to add business identity providers through their metadata documents

- Use the portal to establish claims transformation rules for normalizing the user's attributes

- Do all of the above via management API

- Outsource authentication of a cloud service to ACS

- Use ACS to handle the home realm discovery problem

The Microsoft Azure Access Control Service is a great service to outsource authentication to, as it can easily abstract away the complexity of dealing with multiple business identity providers such as directories enhanced by Active Directory Federation Services or equivalent and even web and social providers such as Windows Live Id, Facebook, Google and Yahoo. Furthermore, ACS offers powerful tools for manipulating the way in which the user's identity is processed before reaching your cloud service.

This intermediate lab gave you a glimpse of the capabilities of ACS applied to classic business access problems such has onboarding new partners, handling authentication from multiple sources and protecting cloud services from changes and edge cases. Here we focused on Websites, but ACS can handle just as well SOAP and REST Web services. We focused on business identity providers, but ACS offers comprehensive support for Web identities via easy to use features which perfectly fit with the agility of Web-based solutions. If you are interested in knowing more about those capabilities, please refer to the introductory hands-on lab. Finally, if you are interested in REST solutions, look out for upcoming new labs exploring the OAuth2 features in ACS.

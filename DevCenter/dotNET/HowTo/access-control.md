<properties linkid="dev-net-how-to-access-control" urlDisplayName="Access Control" pageTitle="How to use Access Control (.NET) - Windows Azure feature guide" metaKeywords="Azure Access Control Service authentication C#" description="Learn how to use Access Control Service (ACS) in your Windows Azure application to authenticate users when they try to gain access to a web app." metaCanonical="" services="active-directory" documentationCenter=".NET" title="How to Authenticate Web Users with Windows Azure Active Directory Access Control" authors=""  solutions="" writer="juneb" manager="" editor=""  />




# How to Authenticate Web Users with Windows Azure Active Directory Access Control


This guide shows you how to use Windows Azure Active Directory Access Control (also known as Access Control Service or ACS) to authenticate users from identity providers such as Microsoft, Google, Yahoo, and Facebook when they try to gain access to a web application.

<h2><span class="short-header">Table of Contents</span>Table of Contents</h2>

-   [What is ACS?][]
-   [Concepts][]
-   [Prerequisites][]
-   [Create an Access Control Namespace][]
-   [Create an ASP.NET MVC Application][]
-   [Integrate your Web Application with ACS][]
-   [Test the Integration with ACS][]
-   [View Claims Sent By ACS][vcsb]
-   [View the Application in the ACS Management Portal][vpp]
-   [Add an Identity Provider][]
-   [What's Next][]

<h2><span class="short-header">What is ACS?</span>What is ACS?</h2>

Most developers are not identity experts and do not want to spend time developing authentication and authorization mechanisms for their applications and services. ACS is a Windows Azure service that provides an easy way for you to authenticate users to access your web applications and services without having to add complex authentication logic to your code.

The following features are available in ACS:

-   Integration with Windows Identity Foundation (WIF).
-   Support for popular web identity providers (IPs) including Microsoft accounts (formerly known as Windows Live ID), Google, Yahoo, and Facebook.
-   Support for Active Directory Federation Services (AD FS) 2.0.
-   An Open Data Protocol (OData)-based management service that provides
    programmatic access to ACS settings.
-   A Management Portal that allows administrative access to the ACS
    settings.

For more information about ACS, see [Access Control Service 2.0][].

<h2><span class="short-header">Concepts</span>Concepts</h2>

ACS is built on the principles of claims-based identity -- a consistent approach to creating authentication mechanisms for applications running on-premises or in the cloud. Claims-based identity provides a common way for applications and services to get the identity information they need about users inside their organization, in other organizations, and on the Internet.

To complete the tasks in this guide, you should understand the following terms and concepts are used in this guide:


**Client** - A browser that is attempting to gain access to your web application.

**Relying party (RP) application** - Your web app. An RP application is a website or service that outsources authentication to one external authority. In identity jargon, we say that the RP trusts that authority. This guide explains how to configure your application to trust ACS.

**Token** - A user gains access to an RP application by presenting a valid token that was issued by an authority that the RP application trusts. A collection of security data that is issued when a client is authenticated. It contains a set of claims, which are attributes of the authenticated user, such as a user's name or age, or an identifier for a user role. A token is digitally signed so its issuer can be identified and its content cannot be changed.

**Identity Provider (IP)** - An  authority that authenticates user identities and issues security tokens, such as Microsoft account (Windows Live ID), Facebook,  Google, Twitter, and Active Directory. When ACS is configured to trust an IP, it accepts and validates the tokens that the IP issues. Because ACS can trust multiple IPs at the same time, when your application trusts ACS, you can  your application can offer users the option to be authenticated by any of the IPs that ACS trusts on your behalf.

**Federation Provider (FP)** - Identity providers (IPs) have direct knowledge of users, authenticate users by using their credentials, and issue claims about users. A Federation Provider (FP) is a different kind of authority. Instead of authenticating users directly, the FP brokers authentication. It acts as an intermediary between a relying party application and one or more IPs. ACS is a federation provider (FP).

**ACS Rule Engine** - Claims transformation rules convert the claims in tokens from trusted IPs so they can be used by an RP. ACS includes a rule engine that  applies the claims transformation rules that you specify for your RP.

**Access Control Namespace** - Provides a unique scope for addressing ACS resources within your application. The namespace contains your settings, such as the IPs you trust, the RP applications you want to serve, the rules that you apply to incoming tokens, and it displays the endpoints that the application and the developer use to communicate with ACS.

The following figure shows how ACS authentication works with a web application:

![][0]

1.  The client (in this case, a browser) requests a page from the RP.
2.  Since the request is not yet authenticated, the RP redirects the
    user to the authority that it trusts, which is ACS. The ACS presents
    the user with the choice of IPs that were specified for this RP. The
    user selects the appropriate IP.
3.  The client browses to the IP's authentication page, and prompts the
    user to log on.
4.  After the client is authenticated (for example, the identity
    credentials are entered), the IP issues a security token.
5.  After issuing a security token, the IP directs the client to send the security token that the IP issued to ACS.
6.  ACS validates the security token issued by the IP, inputs the
    identity claims in this token into the ACS rules engine, calculates
    the output identity claims, and issues a new security token that
    contains these output claims.
7.  ACS directs the client to send the security token that ACS issued to the RP. The RP validates the signature on the security token, extracts claims for use by the application business logic, and returns the page that was originally requested.

<h2><span class="short-header">Prerequisites</span>Prerequisites</h2>


To complete the tasks in this guide, you will need the following:

-	Windows Azure subscription
-	Microsoft Visual Studio 2012 
-	Identity and Access Tool for Visual Studio 2012 (To download, see [Identity and Access Tool][])


<h2><span class="short-header">Create an Access Control Namespace</span>Create an Access Control Namespace</h2>

To use Active Directory Access Control in Windows Azure, create an Access Control namespace. The namespace provides a unique scope for
addressing ACS resources within your application.

1.  Log into the [Windows Azure Management Portal][] (https://manage.WindowsAzure.com).
    
2.  Click **Active Directory**.  

	![][1]

3.  To create a new Access Control namespace, click **New**. **App Services** and **Access Control** will be selected. Click **Quick Create**. 

	![][2]

4.  Enter a name for the namespace. Windows Azure verifies that the name is unique.

5.  Select the region in which the namespace is used. For best performance, use the region in which you are deploying your application, and then click **Create**.

Windows Azure creates and activates the namespace.

<h2><span class="short-header">Create an ASP.NET MVC Application</span>Create an ASP.NET MVC Application</h2>

In this step, you create a ASP.NET MVC application. In later steps, we'll integrate this simple web forms application with ACS.

1.	Start Visual Studio 2012 or Visual Studio Express for Web 2012 (Previous versions of Visual Studio will not work with this tutorial).
1.	Click **File**, and then click **New Project**.
1.	Select the Visual C#/Web template, and then select **ASP.NET MVC 4 Web Application**.

	We'll use a MVC application for this guide, but you can use any web application type for this task.

	![][3]

1. In **Name**, type **MvcACS**, and then click **OK**.
1. In the next dialog, select **Internet Application**, and then click **OK**.
1. Edit the *Views\Shared\_LoginPartial.cshtml* file and replace the contents with the following code:

        @if (Request.IsAuthenticated)
        {
            string name = "Null ID";
            if (!String.IsNullOrEmpty(User.Identity.Name))
            {
                name = User.Identity.Name;
            }    
            <text>
            Hello, @Html.ActionLink(name, "Manage", "Account", routeValues: null, htmlAttributes: new { @class = "username", title = "Manage" })!
                    @using (Html.BeginForm("LogOff", "Account", FormMethod.Post, new { id = "logoutForm" }))
                    {
                        @Html.AntiForgeryToken()
                        <a href="javascript:document.getElementById('logoutForm').submit()">Log off</a>
                    }
            </text>
        }
        else
        {
            <ul>
                <li>@Html.ActionLink("Register", "Register", "Account", routeValues: null, htmlAttributes: new { id = "registerLink" })</li>
                <li>@Html.ActionLink("Log in", "Login", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
            </ul>
        }

Currently, ACS doesn't set User.Identity.Name, so we need to make the above change.

1. Press F5 to run the application. The default ASP.NET MVC application appears in your web browser.

<h2><span class="short-header">Integrate your Web Application with ACS</span>Integrate your Web Application with ACS</h2>

In this task, you will integrate your ASP.NET web application with ACS.

1.	In Solution Explorer, right-click the MvcACS project, and then select **Identity and Access**.

	If the **Identity and Access** option does not appear on the context menu, install the Identity and Access Tool. For information, see [Identity and Access Tool]. 

	![][4]

2.	On the **Providers** tab, select **Use the Windows Azure Access Control Service**.

    ![][44]

3.  Click the **Configure** link.

    ![][444]

	Visual Studio requests information about the Access Control namespace. Enter the namespace name you created earlier (Test in this images above, but you will have a different namespace). Switch back to the Windows Azure Management Portal to get the symmetric key.

	![][17]

4.  In the Windows Azure Management Portal, click the Access Control namespace and then click **Manage**.

	![][8]

5.	Click **Management Service** and then click **Management Client**.

	![][18]

6.	Click **Symmetric Key**, click **Show Key**, and copy the key value. Then, click **Cancel** to exit the Edit Management Client page without making changes. 

	![][19]

7.  In Visual Studio, paste the key in the **Enter the Management Key for the namespace** field and click **Save management key**, and then click **OK**.

	![][20]

	Visual Studio uses the information about the namespace to connect to the ACS Management Portal and get the settings for your namespace, including the identity providers, realm, and return URL.

8.	Select **Windows Live ID** (Microsoft account) and click OK. 

	![][5]

<h2><span class="short-header">Test the Integration with ACS</span>Test the Integration with ACS</h2>

This task explains how to test the integration of your RP application and ACS.

-	Press F5 in Visual Studio to run the app.

When your application is integrated with ACS and you have selected Windows Live ID (Microsoft account), instead of opening the default ASP.NET Web Forms application, your browser is redirected to the sign-in page for Microsoft accounts. When you sign in with a valid user name a password, you are then redirected to the  MvcACS application.

![][6]

Congratulations! You have successfully integrated ACS with your ASP.NET web application. ACS is now handling the authentication of users using their Microsoft account credentials.

<h2><a name="bkmk_viewClaims"></a>View Claims Sent By ACS</h2>

In this section we will modify the application to view the claims sent by ACS.  The Identity and Access tool has created a rule group that passes through all claims from the IP to your application.  Note that different identity providers send different claims.

1. Open the *Controllers\HomeController.cs* file. Add a **using** statement for **System.Threading**:

 	using System.Threading;

1. In the HomeController class add the *Claims* method:

    public ActionResult Claims()
    {
        ViewBag.Message = "Your claims page.";

        ViewBag.ClaimsIdentity = Thread.CurrentPrincipal.Identity;

        return View();
    }

1. Right click on the *Claims* method and select **Add View**.

![][66]

1. Click **Add**.

1. Replace the contents of the *Views\Home\Claims.cshtml* file with the following code:

        @{
            ViewBag.Title = "Claims";
        }
        <hgroup class="title">
            <h1>@ViewBag.Title.</h1>
            <h2>@ViewBag.Message</h2>
        </hgroup>
        <h3>Values from Identity</h3>
        <table>
            <tr>
                <td>
                    IsAuthenticated: 
                </td>
                <td>
                    @ViewBag.ClaimsIdentity.IsAuthenticated 
                </td>
            </tr>
            <tr>
                <td>
                    Name: 
                </td>        
                <td>
                    @ViewBag.ClaimsIdentity.Name
                </td>        
            </tr>
        </table>
        <h3>Claims from ClaimsIdentity</h3>
        <table>
            <tr>
                <th>
                    Claim Type
                </th>
                <th>
                    Claim Value
                </th>
            </tr>
                @foreach (System.Security.Claims.Claim claim in ViewBag.ClaimsIdentity.Claims ) {
            <tr>
                <td>
                    @claim.Type
                </td>
                <td>
                    @claim.Value
                </td>
            </tr>
        }
        </table>

1. Run the application and navigate to the *Claims* method:

![][666]

For more information on using claims in your application, see the [Windows Identity Foundation library](http://msdn.microsoft.com/en-us/library/hh377151.aspx).

<h2><a name="bkmk_VP"></a>View the App in the ACS Management Portal</h2>

The Identity and Access Tool in Visual Studio automatically integrates your application with ACS.

When you select the Use Windows Azure Access Control option and then run your application, the Identity and Access Tool adds your application as a relying party, configures it to use the selected identity providers, and generates and selects the default claims transformation rules for the application.

You can review and change these configuration settings in the ACS Management Portal. Use the following steps to review the changes in the portal.

1.	Log into the Windows [Azure Management Portal](http://manage.WindowsAzure.com).

2.	Click **Active Directory**. 

	![][8]

3.	Select an Access Control namespace and then click **Manage**. This action opens the ACS Management Portal.

	![][9]


4.	Click **Relying party applications**.

	The new MvcACS application appears in the list of relying party applications. The realm is automatically set to the application main page.

	![][10]


5.	Click **MvcACS**.

	The Edit Relying Party Application page contains configuration settings for the MvcACS web application. When you change the settings on this page and save them, the changes are immediately applied to the application.

	![][11]

6.	Scroll down the page to see the remaining configuration settings for the MvcACS application, including the identity providers and claims transformation rules.

	![][12]

In the next section, we'll use the features of the ACS Management Portal to make a change to the web application -- just to show how easy it is to do.

<h2><span class="short-header">Add an Identity Provider</span>Add an Identity Provider</h2>

Let's use the ACS Management Portal to change the authentication of our MvcACS application. In this example, we'll add Google as an identity provider for MvcACS.

1.	Click **Identity providers** (in the navigation menu) and then click **Add**.

	![][13]

2.	Click **Google** and then click **Next**. The MvcACS app checkbox is selected by default. 

	![][14]

3. Click Save. 

	![][15]


Done! If you go back to Visual Studio, open the project for the MvcACS app, and click **Identity and Access**, the tool lists both the Windows Live ID and Google identity providers.  

![][16]

And, when you run your application, you'll see the effect. When an application supports more than one identity provider, the user's browser is first directed to a page hosted by ACS that prompts the user to choose an identity provider. 

![][7]

After the user selects an identity provider, the browser goes to the identity provider sign-in page.

<h2><span class="short-header">What's Next</span>What's Next</h2>

You have created a web application that is integrated with ACS. But, this is just the beginning! You can expand on this scenario.
 
For example, you can add more identity providers for this RP or allow users registered in enterprise directories, such as Active Directory Domain Services, to log on to the web application.

You can also add rules to your namespace that determine which claims are sent to an application for processing in the application business logic.

To further explore ACS functionality and to experiment with more scenarios, see [Access Control Service 2.0].


  [What is ACS?]: #what-is
  [Concepts]: #concepts
  [Prerequisites]: #pre
  [Create an ASP.NET MVC Application]: #create-web-app
  [Create an Access Control Namespace]: #create-namespace
  [Integrate your Web Application with ACS]: #Identity-Access
  [Test the Integration with ACS]: #Test-ACS
  [View the Application in the ACS Management Portal]: acs-portal
  [Add an Identity Provider]: #add-IP
  [What's Next]: #whats-next
  [vcsb]: #bkmk_viewClaims
  [vpp]: #bkmk_VP

  [Access Control Service 2.0]: http://go.microsoft.com/fwlink/?LinkID=212360
  [Identity and Access Tool]: http://go.microsoft.com/fwlink/?LinkID=245849
  [Windows Azure Management Portal]: http://manage.WindowsAzure.com

  [0]: ../../../DevCenter/dotNet/Media/acs-01.png
  [1]: ../../../DevCenter/dotNet/Media/acsCreateNamespace.png
  [2]: ../../../DevCenter/dotNet/Media/acsQuickCreate.png
  [3]: ../../../DevCenter/dotNet/Media/rzMvc.png
  [4]: ../../../DevCenter/dotNet/Media/rzIA.png
[44]: ../../../DevCenter/dotNet/Media/rzPT.png
 [444]: ../../../DevCenter/dotNet/Media/rzC.png
  [5]: ../../../DevCenter/dotNet/Media/acsIdAndAccess1.png
  [6]: ../../../DevCenter/dotNet/Media/acsMSFTAcct.png
  [66]: ../../../DevCenter/dotNet/Media/rzAv.png
  [666]: ../../../DevCenter/dotNet/Media/rzCl.png
  [7]: ../../../DevCenter/dotNet/Media/acsSignIn.png
  [8]: ../../../DevCenter/dotNet/Media/acsClickManage.png
  [9]: ../../../DevCenter/dotNet/Media/acsACSPortal.png
  [10]: ../../../DevCenter/dotNet/Media/acsRPPage.png
  [11]: ../../../DevCenter/dotNet/Media/acsEdit-RP.png
  [12]: ../../../DevCenter/dotNet/Media/acsEdit-RP2.png
  [13]: ../../../DevCenter/dotNet/Media/acsAdd-Idp.png
  [14]: ../../../DevCenter/dotNet/Media/acsAdd-Google.png
  [15]: ../../../DevCenter/dotNet/Media/acsSave-Google.png
  [16]: ../../../DevCenter/dotNet/Media/acsIdAndA-after.png
  [17]: ../../../DevCenter/dotNet/Media/acsConfigAcsNamespace.png
  [18]: ../../../DevCenter/dotNet/Media/acsManagementService.png
  [19]: ../../../DevCenter/dotNet/Media/acsShowKey.png
  [20]: ../../../DevCenter/dotNet/Media/acsConfigAcsNamespace2.png
---
title: 向 ASP.NET Web 应用添加 Microsoft 登录功能 | Microsoft Docs
description: 了解如何使用 OpenID Connect 标准通过基于传统 Web 浏览器的应用程序根据 ASP.NET 解决方案添加 Microsoft 登录。
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
origin.date: 05/21/2019
ms.date: 07/01/2019
ms.author: v-junlch
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2daaa58ee06a7af79bc5c6ef5ffc2a704b39491d
ms.sourcegitcommit: 5f85d6fe825db38579684ee1b621d19b22eeff57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67568737"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-an-aspnet-web-app"></a>快速入门：向 ASP.NET Web 应用添加 Microsoft 登录

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

在本快速入门中，你将了解如何使用 OpenID Connect 通过基于传统 Web 浏览器的应用程序，根据 ASP.NET 模型视图控制器 (MVC) 解决方案实现 Microsoft 登录。 你将了解如何在 ASP.NET 应用程序中使用工作和学校帐户登录。

在本快速入门结束时，应用程序可接受与 Azure Active Directory (Azure AD) 集成的组织的工作和学校帐户登录。

## <a name="prerequisites"></a>先决条件

要开始，请确保满足下列先决条件：

* 安装 Visual Studio 2015 Update 3 或更高版本。 尚未安装？ [免费下载 Visual Studio 2019](https://www.visualstudio.com/downloads/)

## <a name="scenario-sign-in-users-from-work-and-school-accounts-in-your-aspnet-app"></a>方案：在 ASP.NET 应用中让用户使用工作和学校帐户登录

![本指南的工作原理](./media/quickstart-v1-aspnet-webapp/aspnet-intro.png)

浏览器访问 ASP.NET 网站，并请求用户使用此场景中的“登录”按钮进行身份验证。 在此方案中，呈现网页的大部分工作在服务器端完成。

此快速入门演示了如何从空模板起步在 ASP.NET Web 应用程序上让用户登录。 它还包括了添加登录按钮和每个控制器与方法等步骤，并讨论了这些任务背后的概念。 还可以通过使用 [Visual Studio Web 模板](https://docs.microsoft.com/aspnet/visual-studio/overview/2013/creating-web-projects-in-visual-studio#organizational-account-authentication-options)并选择“组织帐户”和云选项之一（该选项使用包含其他控制器、方法和视图的更丰富的模板），创建使 Azure AD 用户（工作和学校帐户）登录的项目  。

## <a name="libraries"></a>库

本快速入门使用以下包：

| 库 | 说明 |
|---|---|
| [Microsoft.Owin.Security.OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) | 让应用程序可使用 OpenIdConnect 进行身份验证的中间件 |
| [Microsoft.Owin.Security.Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies) |让应用程序可使用 Cookie 维持用户会话的中间件 |
| [Microsoft.Owin.Host.SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb) | 让基于 OWIN 的应用程序可使用 ASP.NET 请求管道在 IIS 上运行 |
|  |  |

## <a name="step-1-set-up-your-project"></a>步骤 1：设置项目

这些步骤介绍如何使用 OpenID Connect 通过 OWIN 中间件在 ASP.NET 项目上安装和配置身份验证管道。

要下载此示例的 Visual Studio 项目，请按照下列步骤操作：
1. [下载 GitHub 上的示例项目](https://github.com/AzureADQuickStarts/WebApp-OpenIdConnect-DotNet/archive/GuidedSetup.zip)。
1. 跳至“配置”步骤以在执行操作前配置代码示例。

## <a name="step-2-create-your-aspnet-project"></a>步骤 2：创建 ASP.NET 项目

1. 在 Visual Studio 中，转到“文件”>“新建”>“项目”  。
2. 对于“项目类型”，请选择“Web”，然后选择“ASP.NET Web 应用程序 (.NET Framework)”    。
3. 为应用程序命名，并单击“创建”  。
4. 选择“空”，然后在“添加文件夹和核心引用”下选择“MVC”以添加 MVC 引用    。
5. 选择“创建”  。

## <a name="step-3-add-authentication-components"></a>步骤 3：添加身份验证组件

1. 在 Visual Studio 中，转到“工具”>“NuGet 包管理器”>“包管理器控制台”  。
2. 在包管理器控制台窗口中键入以下命令，添加 **OWIN 中间件 NuGet 包**：

    ```powershell
    Install-Package Microsoft.Owin.Security.OpenIdConnect
    Install-Package Microsoft.Owin.Security.Cookies
    Install-Package Microsoft.Owin.Host.SystemWeb
    ```

<!--start-collapse-->
> ### <a name="about-these-packages"></a>关于这些包
>上述库通过基于 Cookie 的身份验证使用 OpenID Connect 启用单一登录 (SSO)。 完成身份验证后，代表用户的令牌会发送到应用程序，OWIN 中间件会创建会话 Cookie。 浏览器随后对后续请求使用此 cookie，这样一来，用户就无需重新验证，也不需要任何其他验证。
<!--end-collapse-->

## <a name="step-4-configure-the-authentication-pipeline"></a>步骤 4：配置身份验证管道

按照以下步骤创建 OWIN 中间件 Startup 类，以配置 OpenID Connect 身份验证  。 此类自动执行。

> [!TIP]
> 如果项目的根文件夹中没有 `Startup.cs` 文件，请执行以下操作：<br/>
> 1. 右键单击项目的根文件夹：>   “添加”>“新建项”...>“OWIN Startup 类” <br/>
> 2. 将其命名为 `Startup.cs`<br/>
>
>> 确保选择的类是 OWIN Startup 类，而不是标准 C# 类。 通过检查是否在命名空间上看到 `[assembly: OwinStartup(typeof({NameSpace}.Startup))]` 来进行确认。

创建 OWIN 中间件 Startup 类  ：

1. 将 OWIN  和 Microsoft.IdentityModel  命名空间添加到 `Startup.cs`：

    ```C#
    using Microsoft.Owin;
    using Owin;
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    using Microsoft.Owin.Security.Notifications;
    using Microsoft.IdentityModel.Protocols;
    using System;
    using System.Threading.Tasks;
    ```

2. 使用以下代码替换 Startup 类：

    ```C#
    public class Startup
    {
        // The Client ID (a.k.a. Application ID) is used by the application to uniquely identify itself to Azure AD
        string clientId = System.Configuration.ConfigurationManager.AppSettings["ClientId"];

        // RedirectUri is the URL where the user will be redirected to after they sign in
        string redirectUrl = System.Configuration.ConfigurationManager.AppSettings["redirectUrl"];

        // Tenant is the tenant ID (e.g. contoso.partner.onmschina.cn, or 'common' for multi-tenant)
        static string tenant = System.Configuration.ConfigurationManager.AppSettings["Tenant"];

        // Authority is the URL for authority, composed by Azure Active Directory endpoint and the tenant name (e.g. https://login.partner.microsoftonline.cn/contoso.partner.onmschina.cn)
        string authority = String.Format(System.Globalization.CultureInfo.InvariantCulture, System.Configuration.ConfigurationManager.AppSettings["Authority"], tenant);

        /// <summary>
        /// Configure OWIN to use OpenIdConnect 
        /// </summary>
        /// <param name="app"></param>
        public void Configuration(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());
            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {
                    // Sets the ClientId, authority, RedirectUri as obtained from web.config
                    ClientId = clientId,
                    Authority = authority,
                    RedirectUri = redirectUrl,

                    // PostLogoutRedirectUri is the page that users will be redirected to after sign-out. In this case, it is using the home page
                    PostLogoutRedirectUri = redirectUrl,

                    //Scope is the requested scope: OpenIdConnectScopes.OpenIdProfileis equivalent to the string 'openid profile': in the consent screen, this will result in 'Sign you in and read your profile'
                    Scope = OpenIdConnectScopes.OpenIdProfile,

                    // ResponseType is set to request the id_token - which contains basic information about the signed-in user
                    ResponseType = OpenIdConnectResponseTypes.IdToken,

                    // ValidateIssuer set to false to allow work accounts from any organization to sign in to your application
                    // To only allow users from a single organizations, set ValidateIssuer to true and 'tenant' setting in web.config to the tenant name or Id (example: contoso.partner.onmschina.cn)
                    // To allow users from only a list of specific organizations, set ValidateIssuer to true and use ValidIssuers parameter
                    TokenValidationParameters = new System.IdentityModel.Tokens.TokenValidationParameters() { ValidateIssuer = false },

                    // OpenIdConnectAuthenticationNotifications configures OWIN to send notification of failed authentications to OnAuthenticationFailed method
                    Notifications = new OpenIdConnectAuthenticationNotifications
                    {
                        AuthenticationFailed = OnAuthenticationFailed
                    }
                }
            );
        }

        /// <summary>
        /// Handle failed authentication requests by redirecting the user to the home page with an error in the query string
        /// </summary>
        /// <param name="context"></param>
        /// <returns></returns>
        private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> context)
        {
            context.HandleResponse();
            context.Response.Redirect("/?errormessage=" + context.Exception.Message);
            return Task.FromResult(0);
        }
    }
    ```

<!--start-collapse-->
> [!NOTE]
> 在 *OpenIDConnectAuthenticationOptions* 中提供的参数将充当应用程序与 Azure AD 通信时使用的坐标。 OpenID Connect 中间件会使用 Cookie，因此，还需要设置 Cookie 身份验证，如以上代码所示。 *ValidateIssuer* 值告知 OpenIdConnect 不要限制某个特定组织的访问权限。
<!--end-collapse-->

<!--end-setup-->

<!--start-use-->

## <a name="step-5-add-a-controller-to-handle-sign-in-and-sign-out-requests"></a>步骤 5：添加控制器来处理登录和注销请求

创建新控制器来公开登录和注销方法。

1.  右键单击“控制器”文件夹，并选择“添加”>“控制器”  
2.  选择“MVC {version} 控制器 - 空”  。
3.  选择“设置”  （应用程序对象和服务主体对象）。
4.  将其命名为 `HomeController`，然后选择“添加”  。
5.  向该类添加 OWIN  命名空间：

    ```C#
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    ```

6. 通过代码启动身份验证质询，添加下面的方法来处理控制器登录和注销：

    ```C#
    /// <summary>
    /// Send an OpenID Connect sign-in request.
    /// Alternatively, you can just decorate the SignIn method with the [Authorize] attribute
    /// </summary>
    public void SignIn()
    {
        if (!Request.IsAuthenticated)
        {
            HttpContext.GetOwinContext().Authentication.Challenge(
                new AuthenticationProperties { RedirectUri = "/" },
                OpenIdConnectAuthenticationDefaults.AuthenticationType);
        }
    }

    /// <summary>
    /// Send an OpenID Connect sign-out request.
    /// </summary>
    public void SignOut()
    {
        HttpContext.GetOwinContext().Authentication.SignOut(
            OpenIdConnectAuthenticationDefaults.AuthenticationType,
            CookieAuthenticationDefaults.AuthenticationType);
    }
    ```

## <a name="step-6-create-the-apps-home-page-to-sign-in-users-via-a-sign-in-button"></a>步骤 6：创建应用的主页，通过登录按钮来登录用户

在 Visual Studio 中，创建新视图来添加登录按钮并在身份验证后显示用户信息：

1. 右键单击“视图/主页”文件夹，然后选择“添加视图”   。
1. 将其命名为“Index”  。
1. 向文件添加以下 HTML，其中包括登录按钮：

    ```html
    @{
        Layout = null;
    }
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Sign-In with Microsoft Guided Setup (Work Accounts)</title>
    </head>
    <body>
    @if (!Request.IsAuthenticated)
    {
        <!-- If the user is not authenticated, display the sign-in button -->
        <br /><a href="@Url.Action("SignIn", "Home")" style="text-decoration: none;">
            <svg xmlns="http://www.w3.org/2000/svg" xml:space="preserve" width="300px" height="50px" viewBox="0 0 3278 522" class="SignInButton">
                <style type="text/css">.fil0:hover {fill: #4B4B4B;} .fnt0 {font-size: 260px;font-family: 'Segoe UI Semibold', 'Segoe UI'; text-decoration: none;}</style>
                <rect class="fil0" x="2" y="2" width="3174" height="517" fill="black" /><rect x="150" y="129" width="122" height="122" fill="#F35325" /><rect x="284" y="129" width="122" height="122" fill="#81BC06" /><rect x="150" y="263" width="122" height="122" fill="#05A6F0" /><rect x="284" y="263" width="122" height="122" fill="#FFBA08" /><text x="470" y="357" fill="white" class="fnt0">Sign in with Microsoft</text>
            </svg>
        </a>
    }
    else
    {
        <span><br/>Hello @((User.Identity as System.Security.Claims.ClaimsIdentity)?.FindFirst("name")?.Value)</span>
        <br /><br />
        @Html.ActionLink("See Your Claims", "Index", "Claims")
        <br /><br />
        @Html.ActionLink("Sign out", "SignOut", "Home")
    }
    @if (!string.IsNullOrWhiteSpace(Request.QueryString["errormessage"]))
    {
        <div style="background-color:red;color:white;font-weight: bold;">Error: @Request.QueryString["errormessage"]</div>
    }
    </body>
    </html>
    ```

<!--start-collapse-->
此页以 SVG 格式添加登录按钮，背景为黑色：<br/>![使用 Microsoft 登录](./media/quickstart-v1-aspnet-webapp/aspnetsigninbuttonsample.png)<br/> 有关更多登录按钮，请转到[应用程序的品牌指南](howto-add-branding-in-azure-ad-apps.md)。
<!--end-collapse-->

## <a name="step-7-display-users-claims-by-adding-a-controller"></a>步骤 7：添加控制器来显示用户声明

此控制器演示如何使用 `[Authorize]` 属性来保护控制器。 此属性只允许通过身份验证的用户，从而限制对控制器的访问。 下面的代码使用该属性来显示作为登录的一部分被检索的用户声明。

1. 右键单击“控制器”文件夹，然后选择“添加”>“控制器”   。
1. 选择“MVC {version} 控制器 - 空”  。
1. 选择“设置”  （应用程序对象和服务主体对象）。
1. 将其命名为“ClaimsController”  。
1. 将控制器类的代码替换为下面的代码 - 此示例将 `[Authorize]` 属性添加到类：

    ```c#
    [Authorize]
    public class ClaimsController : Controller
    {
        /// <summary>
        /// Add user's claims to viewbag
        /// </summary>
        /// <returns></returns>
        public ActionResult Index()
        {
            var userClaims = User.Identity as System.Security.Claims.ClaimsIdentity;

            //You get the user’s first and last name below:
            ViewBag.Name = userClaims?.FindFirst("name")?.Value;

            // The 'Name' claim can be used for showing the username
            ViewBag.Username = userClaims?.FindFirst(System.IdentityModel.Claims.ClaimTypes.Name)?.Value;

            // The subject/ NameIdentifier claim can be used to uniquely identify the user across the web
            ViewBag.Subject = userClaims?.FindFirst(System.Security.Claims.ClaimTypes.NameIdentifier)?.Value;

            // TenantId is the unique Tenant Id - which represents an organization in Azure AD
            ViewBag.TenantId = userClaims?.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid")?.Value;

            return View();
        }
    }
    ```

<!--start-collapse-->
> [!NOTE]
> 因为使用 `[Authorize]` 属性，仅当用户通过身份验证后，才能执行此控制器的所有方法。 如果用户未通过身份验证，并尝试访问控制器，OWIN 会启动身份验证质询，并强制用户进行身份验证。 上面的代码查看用户令牌中特定属性的用户的声明集合。 这些属性包括用户的全名和用户名，以及全局用户标识符使用者。 它还包含*租户 ID*，表示用户的组织的 ID。
<!--end-collapse-->

## <a name="step-8-create-a-view-to-display-the-users-claims"></a>步骤 8：创建视图来显示用户的声明

在 Visual Studio 中创建新视图，以在网页上显示用户的声明：

1. 右键单击“视图/声明”文件夹，然后选择“添加视图”   。
1. 将其命名为“Index”  。
1. 将以下 HTML 添加到文件：

    ```html
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Sign-In with Microsoft Sample</title>
        <link href="@Url.Content("~/Content/bootstrap.min.css")" rel="stylesheet" type="text/css" />
    </head>
    <body style="padding:50px">
    <h3>Main Claims:</h3>
    <table class="table table-striped table-bordered table-hover">
        <tr><td>Name</td><td>@ViewBag.Name</td></tr>
        <tr><td>Username</td><td>@ViewBag.Username</td></tr>
        <tr><td>Subject</td><td>@ViewBag.Subject</td></tr>
        <tr><td>TenantId</td><td>@ViewBag.TenantId</td></tr>
    </table>
    <br />
    <h3>All Claims:</h3>
    <table class="table table-striped table-bordered table-hover table-condensed">
        @foreach (var claim in ((System.Security.Claims.ClaimsIdentity) User.Identity).Claims)
        {
            <tr><td>@claim.Type</td><td>@claim.Value</td></tr>
        }
    </table>
    <br />
    <br />
    @Html.ActionLink("Sign out", "SignOut", "Home", null, new { @class = "btn btn-primary" })
    </body>
    </html>
    ```

<!--end-use-->

<!--start-configure-->

## <a name="step-9-configure-your-webconfig-and-register-an-application"></a>步骤 9：配置 web.config  并注册应用程序

1. 在 Visual Studio 中，将以下内容添加到 `configuration\appSettings` 部分下的 `web.config`（位于根文件夹中）：

    ```xml
    <add key="ClientId" value="Enter_the_Application_Id_here" />
    <add key="RedirectUrl" value="Enter_the_Redirect_Url_here" />
    <add key="Tenant" value="common" />
    <add key="Authority" value="https://login.partner.microsoftonline.cn/{0}" />
    ```
2. 在解决方案资源管理器中，选择项目并查看“属性”<i></i> 窗口（如果看不到“属性”窗口，请按 F4）
3. 将“已启用 SSL”更改为 <code>True</code>
4. 将项目的 SSL URL 复制到剪贴板：<br/><br/>![项目属性](./media/quickstart-v1-aspnet-webapp/visual-studio-project-properties.png)<br />
5. 在 <code>web.config</code> 中，用项目的 SSL URL替换 <code>Enter_the_Redirect_URL_here</code>。

### <a name="register-your-application-in-the-azure-portal-then-add-its-information-to-webconfig"></a>在 Azure 门户中注册你的应用程序，然后将其信息添加到 *web.config*

1. 使用工作或学校帐户登录到 [Azure 门户](https://portal.azure.cn/)。
2. 如果你的帐户有权访问多个租户，请在右上角选择该帐户，并将门户会话设置为所需的 Azure AD 租户。
3. 导航到面向开发人员的 Microsoft 标识平台的[应用注册](https://portal.azure.cn/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview)页。
4. 选择“新注册”。 
5. “注册应用程序”页显示后，请输入应用程序的名称。 
6. 在“支持的帐户类型”下，选择“任何组织目录中的帐户”。  
7. 在“重定向 URI”  部分下选择  “Web”平台，并将值设置为 Visual Studio 项目的 *SSL URL*（Azure AD 将令牌返回到的位置）。
78. 完成后，选择“注册”  。 在应用的“概述”页上，复制“应用程序(客户端) ID”值。  
9. 返回到 Visual Studio，在 `web.config` 中，用你注册的应用程序 ID 替换 `Enter_the_Application_Id_here`。

> [!TIP]
> 如果帐户配置为可访问多个目录，请确保为要向其注册应用程序的组织选择了正确的目录，方法是单击 Azure 门户右上角的帐户名称，然后按照指示验证所选目录：<br/>![选择正确的目录](./media/quickstart-v1-aspnet-webapp/tenantselector.png)

## <a name="step-10-configure-sign-in-options"></a>步骤 10：配置登录选项

可以将应用程序配置为只允许某个组织的 Azure AD 实例中的用户登录，或者接受任何组织中的用户登录。 按照以下选项之一的说明进行操作：

### <a name="configure-your-application-to-allow-sign-ins-of-work-and-school-accounts-from-any-company-or-organization-multi-tenant"></a>将应用程序配置为允许任何公司或组织（多租户）的工作和学校帐户登录

如果想接受任何已经与 Azure AD 集成的公司或组织的工作和学校帐户登录，请执行以下步骤。 此场景是 *SaaS 应用程序*的常见场景：

1. 返回到 [Azure 门户 - 应用注册](https://portal.azure.cn/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)，找到已注册的应用程序。
2. 在“所有设置”下，选择“属性”   。
3. 将“多租户”属性更改为“是”，然后选择“保存”    。

有关此设置和多租户应用程序概念的详细信息，请参阅[多租户概述](howto-convert-app-to-be-multi-tenant.md)。

### <a name="restrict-users-from-only-one-organizations-active-directory-instance-to-sign-in-to-your-application-single-tenant"></a>限制某个组织的 Active Directory 实例的用户登录应用程序（单租户）

此选项是业务线应用程序的常见方案。

如果希望应用程序仅接受属于特定 Azure AD 实例的帐户（包括该示例的来宾帐户）进行登录，请按照下列步骤操作  ：

1. 使用 `Common` 将 web.config 中的 `Tenant` 参数替换为组织的租户名称 - 例如 contoso.partner.onmschina.cn   。
1. 将 [OWIN Startup 类](#step-4-configure-the-authentication-pipeline)中的 `ValidateIssuer` 参数更改为 `true`  。

要仅允许用户来自特定组织的列表，请按照下列步骤操作：

1. 将 `ValidateIssuer` 设置为 true。
1. 使用 `ValidIssuers` 参数来指定组织列表。

还可通过 IssuerValidator 参数实现自定义方法来验证颁发者  。 有关 `TokenValidationParameters` 的详细信息，请参阅[此 MSDN 文章](https://msdn.microsoft.com/library/system.identitymodel.tokens.tokenvalidationparameters.aspx "TokenValidationParameters MSDN 文章")。

<!--end-configure-->

<!--start-configure-arp-->
<!--
## Configure your ASP.NET Web App with the application's registration information

In this step, you will configure your project to use SSL, and then use the SSL URL to configure your application’s registration information. After this, add the application’ registration information to your solution via *web.config*.

1. In Solution Explorer, select the project and look at the `Properties` window (if you don’t see a Properties window, press F4)
2. Change `SSL Enabled` to `True`
3. Copy the value from `SSL URL` above and paste it in the `Redirect URL` field on the top of this page, then click *Update*:<br/><br/>![Project properties](./media/quickstart-v1-aspnet-webapp/vsprojectproperties.png)<br />
4. Add the following in `web.config` file located in root’s folder, under section `configuration\appSettings`:

```xml
<add key="ClientId" value="[Enter the application Id here]" />
<add key="RedirectUri" value="[Enter the Redirect URL here]" />
<add key="Tenant" value="common" />
<add key="Authority" value="https://login.partner.microsoftonline.cn/{0}" /> 
```
-->
<!--end-configure-arp-->
<!--start-test-->

## <a name="step-11-test-your-code"></a>步骤 11：测试代码

1. 按 F5 在 Visual Studio 中运行项目  。 浏览器随即打开，并定向到 `http://localhost:{port}`，可在其中看到“Microsoft 登录”按钮  。
1. 选择登录按钮。

### <a name="sign-in"></a>登录

准备好测试后，请使用工作帐户 (Azure AD) 登录。

![使用 Microsoft 浏览器窗口登录](./media/quickstart-v1-aspnet-webapp/aspnetbrowsersignin.png)

![使用 Microsoft 浏览器窗口登录](./media/quickstart-v1-aspnet-webapp/aspnetbrowsersignin2.png)

#### <a name="expected-results"></a>预期结果

用户登录后，将被重定向到你的网站主页，即门户上的应用程序注册信息中指定的 HTTPS URL。 此页现在显示“Hello {用户}”、注销链接，以及查看用户声明的链接（即指向之前创建的 Authorize 控制器的链接）  。

### <a name="see-users-claims"></a>查看用户的声明

选择超链接，查看用户的声明。 此操作将用户引至控制器和视图，仅供通过身份验证的用户访问。

#### <a name="expected-results"></a>预期结果

 此时应会显示一个表，其中包含已登录用户的基本属性：

| 属性 | 值 | 说明 |
|---|---|---|
| Name | {用户全名} | 用户的名字和姓氏 |
| 用户名 | <span>user@domain.com</span> | 用于标识已登录用户的用户名 |
| 使用者| {使用者} |一个在 Web 上唯一地标识用户登录名的字符串 |
| 租户 ID | {Guid} | 唯一表示用户的 Azure AD 组织的 guid  |

此外还可看到一个表格，其中包含身份验证请求中的所有用户声明。 有关 ID 令牌和说明中所有声明的列表，请参阅 [ID 令牌中的声明列表](/active-directory/develop/active-directory-token-and-claims)。

### <a name="optional-access-a-method-that-has-an-authorize-attribute"></a>（可选）访问具有 [Authorize] 属性的方法 

此步骤测试作为匿名用户对 Claims 控制器的访问：<br/>
选择注销用户的链接并完成注销过程。<br/>
现在浏览器中键入 `http://localhost:{port}/claims`，访问受 `[Authorize]` 属性保护的控制器

#### <a name="expected-results"></a>预期结果

应会出现提示，要求进行身份验证以查看视图。

## <a name="additional-information"></a>其他信息

<!--start-collapse-->
### <a name="protect-your-entire-web-site"></a>保护整个网站

若要保护整个网站，请在 `Global.asax` `Application_Start` 方法中将 `AuthorizeAttribute` 添加到 `GlobalFilters`：

```csharp
GlobalFilters.Filters.Add(new AuthorizeAttribute());
```
<!--end-collapse-->

<div></div>
<br/>

<!--end-test-->

## <a name="next-steps"></a>后续步骤

现在，可以转到其他方案。

> [!div class="nextstepaction"]
> [ASP.NET 教程](/active-directory/develop/tutorial-v2-asp-webapp)


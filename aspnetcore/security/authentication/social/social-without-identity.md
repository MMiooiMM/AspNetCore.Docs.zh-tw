---
title: 不 ASP.NET Core 身分識別的 Facebook、Google 及外部提供者驗證
author: rick-anderson
description: 使用 Facebook、Google、Twitter 等的說明，而不 ASP.NET Core 身分識別的帳戶使用者驗證。
ms.author: riande
ms.date: 09/25/2019
uid: security/authentication/social/social-without-identity
ms.openlocfilehash: 54dd93a13b2f7ed09c2c305f529d5f4610567184
ms.sourcegitcommit: 6d26ab647ede4f8e57465e29b03be5cb130fc872
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/07/2019
ms.locfileid: "71999893"
---
# <a name="use-social-sign-in-provider-authentication-without-aspnet-core-identity"></a>使用不含 ASP.NET Core 身分識別的社交登入提供者驗證

::: moniker range=">= aspnetcore-3.0"

<xref:security/authentication/social/index> 說明如何讓使用者使用 OAuth 2.0 搭配外部驗證提供者的認證來登入。 該主題中所述的方法包含 ASP.NET Core 身分識別做為驗證提供者。

這個範例示範如何使用**沒有**ASP.NET Core 身分識別的外部驗證提供者。 這適用于不需要所有 ASP.NET Core 身分識別功能的應用程式，但仍需要與信任的外部驗證提供者整合。

這個範例會使用[Google 驗證](xref:security/authentication/google-logins)來驗證使用者。 使用 Google authentication 會將管理登入程式的許多複雜性轉移到 Google。 若要與不同的外部驗證提供者整合，請參閱下列主題：

* [Facebook 驗證](xref:security/authentication/facebook-logins)
* [Microsoft 驗證](xref:security/authentication/microsoft-logins)
* [Twitter 驗證](xref:security/authentication/twitter-logins)
* [其他提供者](xref:security/authentication/otherlogins)

## <a name="configuration"></a>組態

在 `ConfigureServices` 方法中，使用 <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*>、<xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> 和 @no__t 3 方法來設定應用程式的驗證配置：

[!code-csharp[](social-without-identity/3.0sample/Startup.cs?name=snippet1)]

@No__t-0 的呼叫會設定應用程式的 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme>。 @No__t-0 是下列 @no__t 1 驗證擴充方法所使用的預設配置：

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

將應用程式的 `DefaultScheme` 設定為[CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) （"Cookies"），會將應用程式設為使用 cookie 做為這些擴充方法的預設配置。 將應用程式的 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> 設為[GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) （"Google"），會將應用程式設定為使用 Google 做為呼叫 `ChallengeAsync` 的預設配置。 `DefaultChallengeScheme` 會覆寫 `DefaultScheme`。 如需在設定時覆寫 `DefaultScheme` 的其他屬性，請參閱 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions>。

在 `Startup.Configure` 中，呼叫 `UseAuthentication` 並 `UseAuthorization` 來設定 `HttpContext.User` 屬性，並針對要求執行授權中介軟體。 在呼叫 `UseEndpoints` 之前，請先呼叫 `UseAuthentication` 和 @no__t 1 方法：

[!code-csharp[](social-without-identity/3.0sample/Startup.cs?name=snippet2)]

若要深入瞭解驗證配置和 cookie 驗證，請參閱 <xref:security/authentication/cookie>。

## <a name="apply-authorization"></a>套用授權

藉由將 `AuthorizeAttribute` 屬性套用至控制器、動作或頁面，測試應用程式的驗證設定。 下列程式碼會將*隱私權*頁面的存取限制為已驗證的使用者：

[!code-csharp[](social-without-identity/3.0sample/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>登出

若要登出目前的使用者並刪除其 cookie，請呼叫[SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*)。 下列程式碼會在 [*索引*] 頁面中加入 `Logout` 頁面處理常式：

[!code-csharp[](social-without-identity/3.0sample/Pages/Index.cshtml.cs?name=snippet&highlight=14-18)]

請注意，對 `SignOutAsync` 的呼叫並未指定驗證配置。 應用程式的 `DefaultScheme` （`CookieAuthenticationDefaults.AuthenticationScheme`）會用來回複。

## <a name="additional-resources"></a>其他資源

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
::: moniker range="< aspnetcore-3.0"

<xref:security/authentication/social/index> 說明如何讓使用者使用 OAuth 2.0 搭配外部驗證提供者的認證來登入。 該主題中所述的方法包含 ASP.NET Core 身分識別做為驗證提供者。

這個範例示範如何使用**沒有**ASP.NET Core 身分識別的外部驗證提供者。 這適用于不需要所有 ASP.NET Core 身分識別功能的應用程式，但仍需要與信任的外部驗證提供者整合。

這個範例會使用[Google 驗證](xref:security/authentication/google-logins)來驗證使用者。 使用 Google authentication 會將管理登入程式的許多複雜性轉移到 Google。 若要與不同的外部驗證提供者整合，請參閱下列主題：

* [Facebook 驗證](xref:security/authentication/facebook-logins)
* [Microsoft 驗證](xref:security/authentication/microsoft-logins)
* [Twitter 驗證](xref:security/authentication/twitter-logins)
* [其他提供者](xref:security/authentication/otherlogins)

## <a name="configuration"></a>組態

在 `ConfigureServices` 方法中，使用 `AddAuthentication`、`AddCookie` 和 @no__t 3 方法來設定應用程式的驗證配置：

[!code-csharp[](social-without-identity/sample/Startup.cs?name=snippet1)]

對[AddAuthentication](/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication#Microsoft_Extensions_DependencyInjection_AuthenticationServiceCollectionExtensions_AddAuthentication_Microsoft_Extensions_DependencyInjection_IServiceCollection_System_Action_Microsoft_AspNetCore_Authentication_AuthenticationOptions__)的呼叫會設定應用程式的[DefaultScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme)。 @No__t-0 是下列 @no__t 1 驗證擴充方法所使用的預設配置：

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

將應用程式的 `DefaultScheme` 設定為[CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) （"Cookies"），會將應用程式設為使用 cookie 做為這些擴充方法的預設配置。 將應用程式的 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> 設為[GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) （"Google"），會將應用程式設定為使用 Google 做為呼叫 `ChallengeAsync` 的預設配置。 `DefaultChallengeScheme` 會覆寫 `DefaultScheme`。 如需在設定時覆寫 `DefaultScheme` 的其他屬性，請參閱 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions>。

在 `Configure` 方法中，呼叫 `UseAuthentication` 方法，以叫用設定 `HttpContext.User` 屬性的驗證中介軟體。 在呼叫 `UseMvcWithDefaultRoute` 或 `UseMvc` 之前，請先呼叫 `UseAuthentication` 方法：

[!code-csharp[](social-without-identity/sample/Startup.cs?name=snippet2)]

若要深入瞭解驗證配置和 cookie 驗證，請參閱 <xref:security/authentication/cookie>。

## <a name="apply-authorization"></a>套用授權

藉由將 `AuthorizeAttribute` 屬性套用至控制器、動作或頁面，測試應用程式的驗證設定。 下列程式碼會將*隱私權*頁面的存取限制為已驗證的使用者：

[!code-csharp[](social-without-identity/sample/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>登出

若要登出目前的使用者並刪除其 cookie，請呼叫[SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*)。 下列程式碼會在 [*索引*] 頁面中加入 `Logout` 頁面處理常式：

[!code-csharp[](social-without-identity/sample/Pages/Index.cshtml.cs?name=snippet&highlight=7-11)]

請注意，對 `SignOutAsync` 的呼叫並未指定驗證配置。 應用程式的 `DefaultScheme` （`CookieAuthenticationDefaults.AuthenticationScheme`）會用來回複。

## <a name="additional-resources"></a>其他資源

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end

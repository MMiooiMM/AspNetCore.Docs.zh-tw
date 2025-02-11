---
title: ASP.NET Core SignalR 中的安全性考慮
author: bradygaster
description: 瞭解如何在 ASP.NET Core SignalR 中使用驗證和授權。
monikerRange: '>= aspnetcore-2.1'
ms.author: anurse
ms.custom: mvc
ms.date: 11/06/2018
uid: signalr/security
ms.openlocfilehash: a52db2ff51c55f7299d63aa3c7398f99727e0694
ms.sourcegitcommit: 387cf29f5d5addef2cbc70670a11d612806b36b2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2019
ms.locfileid: "70746550"
---
# <a name="security-considerations-in-aspnet-core-signalr"></a>ASP.NET Core SignalR 中的安全性考慮

[Andrew Stanton-護士](https://twitter.com/anurse)

本文提供保護 SignalR 的相關資訊。

## <a name="cross-origin-resource-sharing"></a>跨原始來源資源分享

[跨原始來源資源分享（CORS）](https://www.w3.org/TR/cors/)可以用來允許瀏覽器中的跨原始來源 SignalR 連接。 如果 JavaScript 程式碼裝載于與 SignalR 應用程式不同的網域，則必須啟用[CORS 中介軟體](xref:security/cors)，才能允許 JavaScript 連線到 SignalR 應用程式。 只允許來自您信任或控制之網域的跨原始來源要求。 例如：

* 您的網站託管于`http://www.example.com`
* 您的 SignalR 應用程式託管于`http://signalr.example.com`

應該在 SignalR 應用程式中設定 CORS，以便只允許來源`www.example.com`。

如需設定 CORS 的詳細資訊，請參閱[啟用跨原始來源要求（CORS）](xref:security/cors)。 SignalR**需要**下列 CORS 原則：

* 允許特定的預期來源。 允許任何來源，**但不是安全或**建議的。
* HTTP 方法`GET`和`POST`必須是允許的。
* 即使未使用驗證，也必須啟用認證。

例如，下列 CORS 原則允許裝載在上`https://example.com`的 SignalR 瀏覽器用戶端存取裝載于`https://signalr.example.com`的 SignalR 應用程式：

::: moniker range=">= aspnetcore-3.0"

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    // ... other middleware ...

    // Make sure the CORS middleware is ahead of SignalR.
    app.UseCors(builder =>
    {
        builder.WithOrigins("https://example.com")
            .AllowAnyHeader()
            .WithMethods("GET", "POST")
            .AllowCredentials();
    });

    // ... other middleware ...
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chatHub");
    });

    // ... other middleware ...
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Main](security/sample/Startup.cs?name=snippet1)]

::: moniker-end

> [!NOTE]
> SignalR 與 Azure App Service 中的內建 CORS 功能不相容。

## <a name="websocket-origin-restriction"></a>WebSocket 來源限制

::: moniker range=">= aspnetcore-2.2"

CORS 所提供的保護不套用至 WebSocket。 如需 Websocket 的原始限制，請參閱[websocket 原始限制](xref:fundamentals/websockets#websocket-origin-restriction)。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

CORS 所提供的保護不套用至 WebSocket。 瀏覽器**不**會：

* 執行 CORS 的事前要求。
* 進行 WebSocket 要求時，採用 `Access-Control` 標頭中所指定的限制。

不過，瀏覽器會在發出 WebSocket 要求時，傳送 `Origin` 標頭。 應設定應用程式驗證這些標頭，以確保只允許來自預期來源的 WebSocket。

在 ASP.NET Core 2.1 和更新版本中，可以使用之前`Configure`  **`UseSignalR`** 放置的自訂中介軟體來達成標頭驗證，並在中進行驗證中介軟體：

[!code-csharp[Main](security/sample/Startup.cs?name=snippet2)]

> [!NOTE]
> 因為 `Origin` 由用戶端控制，所以和 `Referer` 標頭一樣可能受到偽造。 這些標頭**不**應做為驗證機制使用。

::: moniker-end

## <a name="access-token-logging"></a>存取權杖記錄

使用 Websocket 或伺服器傳送事件時，瀏覽器用戶端會在查詢字串中傳送存取權杖。 透過查詢字串接收存取權杖通常與使用標準`Authorization`標頭一樣安全。 您應該一律使用 HTTPS 來確保用戶端與伺服器之間的安全端對端連接。 許多 web 伺服器會記錄每個要求的 URL，包括查詢字串。 記錄 Url 可能會記錄存取權杖。 ASP.NET Core 預設會記錄每個要求的 URL，其中會包含查詢字串。 例如：

```
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/myhub?access_token=1234
```

如果您對於使用伺服器記錄檔記錄這項資料有疑慮，您可以將`Microsoft.AspNetCore.Hosting`記錄器設定`Warning`為層級（或更新版本）來完全停用此記錄（ `Info`這些訊息是在層級寫入）。 如需詳細資訊，請參閱[記錄檔篩選](xref:fundamentals/logging/index#log-filtering)的相關檔。 如果您仍然想要記錄特定的要求資訊，可以[撰寫中介軟體](xref:fundamentals/middleware/write)來記錄所需的資料，並篩選出`access_token`查詢字串值（如果有的話）。

## <a name="exceptions"></a>例外狀況

例外狀況訊息通常會被視為不應該向用戶端顯示的敏感性資料。 根據預設，SignalR 不會將中樞方法擲回之例外狀況的詳細資料傳送給用戶端。 相反地，用戶端會收到一般訊息，指出發生錯誤。 可以使用[`EnableDetailedErrors`](xref:signalr/configuration#configure-server-options)覆寫傳遞給用戶端的例外狀況訊息（例如在開發或測試中）。 例外狀況訊息不應在生產環境應用程式中公開給用戶端。

## <a name="buffer-management"></a>緩衝區管理

SignalR 會使用每個連接的緩衝區來管理傳入和傳出訊息。 根據預設，SignalR 會將這些緩衝區限制為 32 KB。 用戶端或伺服器可以傳送的最大訊息為 32 KB。 訊息的連接所耗用的最大記憶體為 32 KB。 如果您的訊息一律小於 32 KB，您可以減少限制，這會：

* 防止用戶端能夠傳送較大的訊息。
* 伺服器永遠都不需要配置大型緩衝區來接受訊息。

如果您的訊息大於 32 KB，您可以增加限制。 增加此限制表示：

* 用戶端可能會導致伺服器配置大量的記憶體緩衝區。
* 大型緩衝區的伺服器配置可能會減少並行連接的數目。

傳入和傳出訊息都有限制，這兩者都可以在中[`HttpConnectionDispatcherOptions`](xref:signalr/configuration#configure-server-options) `MapHub`設定的物件上進行設定：

* `ApplicationMaxBufferSize`代表用戶端中伺服器緩衝區的最大位元組數目。 如果用戶端嘗試傳送大於此限制的訊息，連接可能會關閉。
* `TransportMaxBufferSize`表示伺服器可以傳送的最大位元組數。 如果伺服器嘗試傳送大於此限制的訊息（包括來自中樞方法的傳回值），則會擲回例外狀況。

將限制設為`0`會停用限制。 移除此限制可讓用戶端傳送任何大小的訊息。 傳送大型訊息的惡意用戶端可能會導致配置過量的記憶體。 過多的記憶體使用量可能會大幅減少並行連接的數目。

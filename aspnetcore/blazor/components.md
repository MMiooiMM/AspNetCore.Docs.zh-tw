---
title: 建立和使用 ASP.NET Core Razor 元件
author: guardrex
description: 瞭解如何建立和使用 Razor 元件，包括如何系結至資料、處理事件，以及管理元件生命週期。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/20/2019
uid: blazor/components
ms.openlocfilehash: 065a3a078c56f813ed38f85d7414f22061217dff
ms.sourcegitcommit: eb4fcdeb2f9e8413117624de42841a4997d1d82d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/21/2019
ms.locfileid: "72697954"
---
# <a name="create-and-use-aspnet-core-razor-components"></a>建立和使用 ASP.NET Core Razor 元件

By [Luke Latham](https://github.com/guardrex)和[Daniel Roth](https://github.com/danroth27)

[檢視或下載範例程式碼](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) \(英文\) ([如何下載](xref:index#how-to-download-a-sample))

Blazor 應用程式是使用*元件*所建立。 「元件」（component）是一種獨立的使用者介面（UI）區塊，例如頁面、對話方塊或表單。 元件包含 HTML 標籤，以及插入資料或回應 UI 事件所需的處理邏輯。 元件具有彈性且輕量。 它們可以在專案之間進行嵌套、重複使用及共用。

## <a name="component-classes"></a>元件類別

元件會使用C#和 HTML 標籤的組合，在[razor](xref:mvc/views/razor)元件檔案（*razor*）中執行。 Blazor 中的元件正式稱為*Razor 元件*。

元件的名稱必須以大寫字元開頭。 例如， *MyCoolComponent*有效，而*MyCoolComponent*則無效。

元件的 UI 是使用 HTML 定義的。 動態轉譯邏輯 (例如迴圈、條件、運算式) 是使用內嵌的 C# 語法 (稱為 [Razor](xref:mvc/views/razor)) 來新增的。 編譯應用程式時，會將 HTML 標籤和C#轉譯邏輯轉換成元件類別。 產生的類別名稱與檔案的名稱相符。

元件類別的成員均定義於 `@code` 區塊中。 在 `@code` 區塊中，元件狀態（屬性、欄位）是以事件處理或定義其他元件邏輯的方法來指定。 允許一個以上的 `@code` 區塊。

> [!NOTE]
> 在 ASP.NET Core 3.0 的先前預覽中，`@functions` 組塊用於與 Razor 元件中 `@code` 組塊相同的用途。 `@functions` 區塊會繼續在 Razor 元件中運作，但我們建議您在 ASP.NET Core 3.0 Preview 6 或更新版本中使用 `@code` 區塊。

元件成員可以使用C#開頭為 `@` 的運算式，做為元件轉譯邏輯的一部分。 例如， C#欄位的呈現方式是在功能變數名稱前面加上 `@`。 下列範例會評估並呈現：

* `_headingFontStyle` 至 `font-style` 的 CSS 屬性值。
* `_headingText` 至 `<h1>` 元素的內容。

```cshtml
<h1 style="font-style:@_headingFontStyle">@_headingText</h1>

@code {
    private string _headingFontStyle = "italic";
    private string _headingText = "Put on your new Blazor!";
}
```

一開始呈現元件之後，元件會重新產生其轉譯樹狀結構，以回應事件。 然後，Blazor 會比較新的轉譯樹狀結構與上一個，並將任何修改套用至瀏覽器的檔物件模型（DOM）。

元件是一般C#類別，可以放在專案內的任何位置。 產生網頁的元件通常會位於*Pages*資料夾中。 非頁面元件通常會放在*共用*資料夾中，或加入至專案的自訂資料夾中。 若要使用自訂資料夾，請將自訂資料夾的命名空間新增至父元件或應用程式的 *_Imports*檔案。 例如，下列命名空間會在應用程式的根命名空間為 `WebApplication` 時，讓*元件*資料夾中的元件可供使用：

```cshtml
@using WebApplication.Components
```

## <a name="integrate-components-into-razor-pages-and-mvc-apps"></a>將元件整合到 Razor Pages 和 MVC 應用程式

將元件與現有的 Razor Pages 和 MVC 應用程式搭配使用。 不需要重新撰寫現有的頁面或 views 就能使用 Razor 元件。 當頁面或視圖呈現時，會同時資源清單元件。

若要從頁面或視圖呈現元件，請使用 `RenderComponentAsync<TComponent>` HTML helper 方法：

```cshtml
<div id="MyComponent">
    @(await Html.RenderComponentAsync<MyComponent>(RenderMode.ServerPrerendered))
</div>
```

雖然頁面和視圖可以使用元件，但相反的情況並非如此。 元件不能使用視圖和頁面特定的案例，例如部分視圖和區段。 若要在元件中使用部分視圖的邏輯，請將部分視圖邏輯分解成元件。

如需有關如何呈現元件，以及如何在 Blazor Server 應用程式中管理元件狀態的詳細資訊，請參閱 <xref:blazor/hosting-models> 文章。

## <a name="use-components"></a>使用元件

元件可以包含其他元件，方法是使用 HTML 專案語法來宣告它們。 使用元件的標記看起來像是 HTML 標籤，其中標籤名稱是元件類型。

屬性系結會區分大小寫。 例如，`@bind` 有效，而 `@Bind` 則無效。

在*Index*中的下列標記會轉譯 `HeadingComponent` 實例：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/Index.razor?name=snippet_HeadingComponent)]

*Components/HeadingComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/HeadingComponent.razor)]

如果元件包含的 HTML 專案具有大寫的第一個字母，但不符合元件名稱，則會發出警告，指出該元素有未預期的名稱。 為元件的命名空間加入 `@using` 語句，會使元件可供使用，這會移除警告。

## <a name="component-parameters"></a>元件參數

元件可以具有*元件參數*，其使用元件類別上的公用屬性（具有 `[Parameter]` 屬性）來定義。 使用這些屬性來指定標記中元件的引數。

*Components/ChildComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=11-12)]

在下列範例中，`ParentComponent` 會設定 `ChildComponent` 之 `Title` 屬性的值。

*Pages/ParentComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=5-6)]

## <a name="child-content"></a>子內容

元件可以設定另一個元件的內容。 指派元件會在指定接收元件的標記之間提供內容。

在下列範例中，`ChildComponent` 具有代表 `RenderFragment` 的 `ChildContent` 屬性，代表要呈現的 UI 區段。 @No__t_0 的值位於元件的標記中，應在其中呈現內容。 從父元件接收 `ChildContent` 的值，並在啟動載入面板的 `panel-body` 中轉譯。

*Components/ChildComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> 接收 `RenderFragment` 內容的屬性必須依照慣例命名為 `ChildContent`。

下列 `ParentComponent` 可以提供內容來呈現 `ChildComponent`，方法是將內容放在 `<ChildComponent>` 標記內。

*Pages/ParentComponent. razor*：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=7-8)]

## <a name="attribute-splatting-and-arbitrary-parameters"></a>屬性展開和任意參數

除了元件的宣告參數之外，元件還可以捕捉和轉譯其他屬性。 您可以在字典中捕捉其他屬性，然後在使用[@attributes](xref:mvc/views/razor#attributes) Razor 指示詞轉譯元件時， *splatted*至元素。 當定義的元件會產生支援各種自訂的標記專案時，這個案例就很有用。 例如，針對支援許多參數的 `<input>` 分別定義屬性，可能會很繁瑣。

在下列範例中，第一個 `<input>` 元素（`id="useIndividualParams"`）會使用個別的元件參數，而第二個 `<input>` 元素（`id="useAttributesDict"`）則使用屬性展開：

```cshtml
<input id="useIndividualParams"
       maxlength="@Maxlength"
       placeholder="@Placeholder"
       required="@Required"
       size="@Size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    [Parameter]
    public string Maxlength { get; set; } = "10";

    [Parameter]
    public string Placeholder { get; set; } = "Input placeholder text";

    [Parameter]
    public string Required { get; set; } = "required";

    [Parameter]
    public string Size { get; set; } = "50";

    [Parameter]
    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

參數的類型必須使用字串索引鍵來執行 `IEnumerable<KeyValuePair<string, object>>`。 在此案例中，使用 `IReadOnlyDictionary<string, object>` 也是一個選項。

使用這兩種方法轉譯的 `<input>` 元素相同：

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

若要接受任意屬性，請使用 `[Parameter]` 屬性定義元件參數，並將 `CaptureUnmatchedValues` 屬性設定為 `true`：

```cshtml
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

@No__t_1 上的 `CaptureUnmatchedValues` 屬性可讓參數符合所有不符合任何其他參數的屬性。 元件只能定義具有 `CaptureUnmatchedValues` 的單一參數。 搭配 `CaptureUnmatchedValues` 使用的屬性類型必須可從具有字串索引鍵的 `Dictionary<string, object>` 指派。 `IEnumerable<KeyValuePair<string, object>>` 或 `IReadOnlyDictionary<string, object>` 也是此案例中的選項。

## <a name="data-binding"></a>資料繫結

元件和 DOM 元素的資料系結都是使用[@bind](xref:mvc/views/razor#bind)屬性來完成。 下列範例會將 `CurrentValue` 屬性系結至文字方塊的值：

```cshtml
<input @bind="CurrentValue" />

@code {
    private string CurrentValue { get; set; }
}
```

當文字方塊失去焦點時，就會更新屬性的值。

只有在呈現元件時，才會更新 UI 中的文字方塊，而不會回應變更屬性的值。 由於元件會在事件處理常式程式碼執行之後自行呈現，因此在觸發事件處理常式之後，屬性更新*通常*會立即反映在 UI 中。

使用 `@bind` 搭配 `CurrentValue` 屬性（`<input @bind="CurrentValue" />`）基本上等同于下列內容：

```cshtml
<input value="@CurrentValue"
    @onchange="@((ChangeEventArgs __e) => CurrentValue = 
        __e.Value.ToString())" />
        
@code {
    private string CurrentValue { get; set; }
}
```

當元件呈現時，輸入專案的 `value` 會來自 `CurrentValue` 屬性。 當使用者在文字方塊中輸入並變更元素焦點時，`onchange` 事件會引發，而 `CurrentValue` 屬性會設定為變更的值。 實際上，程式碼產生更為複雜，因為 `@bind` 會處理類型轉換執行的情況。 就原則而言，`@bind` 會將運算式的目前值與 `value` 屬性產生關聯，並使用已註冊的處理常式來處理變更。

除了使用 `@bind` 語法處理 `onchange` 事件之外，也可以使用其他事件來系結屬性或欄位，方法是指定具有 `event` 參數（[@bind-value:event](xref:mvc/views/razor#bind)）的[@bind-value](xref:mvc/views/razor#bind)屬性。 下列範例會系結 `oninput` 事件的 `CurrentValue` 屬性：

```cshtml
<input @bind-value="CurrentValue" @bind-value:event="oninput" />

@code {
    private string CurrentValue { get; set; }
}
```

不同于當元素失去焦點時引發的 `onchange`，`oninput` 會在文字方塊的值變更時引發。

**無法剖析的值**

當使用者將無法剖析的值提供給資料系結元素時，會在觸發系結事件時，自動將無法剖析的值還原成先前的值。

請考慮下列案例：

* @No__t_0 元素會系結至 `int` 類型，其初始值為 `123`：

  ```cshtml
  <input @bind="MyProperty" />

  @code {
      [Parameter]
      public int MyProperty { get; set; } = 123;
  }
  ```
* 使用者會將專案的值更新為頁面中的 `123.45`，並變更元素的焦點。

在上述案例中，元素的值會還原為 `123`。 當 `123.45` 的值拒絕 `123` 的原始值時，使用者瞭解其值不被接受。

根據預設，系結會套用至元素的 `onchange` 事件（`@bind="{PROPERTY OR FIELD}"`）。 使用 `@bind-value="{PROPERTY OR FIELD}" @bind-value:event={EVENT}` 來設定不同的事件。 若為 `oninput` 事件（`@bind-value:event="oninput"`），回復會在引進無法剖析值的任何擊鍵之後發生。 以 `int` 系結類型的 `oninput` 事件為目標時，使用者無法輸入 `.` 字元。 會立即移除 `.` 字元，因此使用者會立即收到僅允許整數的意見反應。 在某些情況下，在 `oninput` 事件上還原值不是理想的情況，例如當使用者應允許清除無法剖析的 `<input>` 值時。 替代方案包括：

* 請勿使用 `oninput` 事件。 使用預設的 `onchange` 事件（`@bind="{PROPERTY OR FIELD}"`），在此情況下，除非元素失去焦點，否則不正確值不會還原。
* 系結至可為 null 的型別（例如 `int?` 或 `string`），並提供自訂邏輯來處理不正確專案。
* 使用[表單驗證元件](xref:blazor/forms-validation)，例如 `InputNumber` 或 `InputDate`。 表單驗證元件有內建支援，可管理不正確輸入。 表單驗證元件：
  * 允許使用者在相關聯的 `EditContext` 上提供不正確輸入和接收驗證錯誤。
  * 在 UI 中顯示驗證錯誤，而不幹擾使用者輸入其他 webform 資料。

**全球化**

`@bind` 值會格式化為使用目前文化特性的規則來顯示和剖析。

您可以從 <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> 屬性存取目前的文化特性。

[InvariantCulture](xref:System.Globalization.CultureInfo.InvariantCulture)用於下欄欄位類型（`<input type="{TYPE}" />`）：

* `date`
* `number`

前面的欄位類型：

* 會使用其適當的瀏覽器格式規則來顯示。
* 不能包含自由格式的文字。
* 根據瀏覽器的執行方式提供使用者互動特性。

下欄欄位類型具有特定的格式需求，而且目前不受 Blazor 支援，因為並非所有主要瀏覽器都支援：

* `datetime-local`
* `month`
* `week`

`@bind` 支援 `@bind:culture` 參數，以提供用來剖析和格式化值的 <xref:System.Globalization.CultureInfo?displayProperty=fullName>。 使用 `date` 和 `number` 欄位類型時，不建議指定文化特性。 `date` 和 `number` 具有內建的 Blazor 支援，可提供所需的文化特性。

如需如何設定使用者文化特性的詳細資訊，請參閱[當地語系化](#localization)一節。

**格式字串**

資料系結搭配使用[@bind:format](xref:mvc/views/razor#bind)的 <xref:System.DateTime> 格式字串。 目前無法使用其他格式運算式，例如貨幣或數位格式。

```cshtml
<input @bind="StartDate" @bind:format="yyyy-MM-dd" />

@code {
    [Parameter]
    public DateTime StartDate { get; set; } = new DateTime(2020, 1, 1);
}
```

在上述程式碼中，`<input>` 元素的欄位類型（`type`）預設為 `text`。 支援下列 .NET 類型的 `@bind:format`：

* <xref:System.DateTime?displayProperty=fullName>
* <xref:System.DateTime?displayProperty=fullName>?
* <xref:System.DateTimeOffset?displayProperty=fullName>
* <xref:System.DateTimeOffset?displayProperty=fullName>?

@No__t_0 屬性會指定要套用至 `<input>` 元素 `value` 的日期格式。 當發生 `onchange` 事件時，也會使用格式來剖析值。

不建議指定 `date` 欄位類型的格式，因為 Blazor 具有格式日期的內建支援。

**元件參數**

Binding 可辨識元件參數，其中 `@bind-{property}` 可以跨元件系結屬性值。

下列子元件（`ChildComponent`）具有 `Year` 元件參數和 `YearChanged` 回呼：

```cshtml
<h2>Child Component</h2>

<p>Year: @Year</p>

@code {
    [Parameter]
    public int Year { get; set; }

    [Parameter]
    public EventCallback<int> YearChanged { get; set; }
}
```

`EventCallback<T>` 會在[app eventcallback](#eventcallback)一節中說明。

下列父元件會使用 `ChildComponent`，並將 `ParentYear` 參數從父系系結至子元件上的 `Year` 參數：

```cshtml
@page "/ParentComponent"

<h1>Parent Component</h1>

<p>ParentYear: @ParentYear</p>

<ChildComponent @bind-Year="ParentYear" />

<button class="btn btn-primary" @onclick="ChangeTheYear">
    Change Year to 1986
</button>

@code {
    [Parameter]
    public int ParentYear { get; set; } = 1978;

    private void ChangeTheYear()
    {
        ParentYear = 1986;
    }
}
```

載入 `ParentComponent` 會產生下列標記：

```html
<h1>Parent Component</h1>

<p>ParentYear: 1978</p>

<h2>Child Component</h2>

<p>Year: 1978</p>
```

如果 `ParentYear` 屬性的值是藉由選取 `ParentComponent` 中的按鈕來變更，則會更新 `ChildComponent` 的 `Year` 屬性。 重新顯示 `ParentComponent` 時，會在 UI 中轉譯 `Year` 的新值：

```html
<h1>Parent Component</h1>

<p>ParentYear: 1986</p>

<h2>Child Component</h2>

<p>Year: 1986</p>
```

@No__t_0 參數是可系結的，因為它具有符合 `Year` 參數類型的伴隨 `YearChanged` 事件。

依照慣例，`<ChildComponent @bind-Year="ParentYear" />` 基本上等同于撰寫：

```cshtml
<ChildComponent @bind-Year="ParentYear" @bind-Year:event="YearChanged" />
```

一般來說，您可以使用 `@bind-property:event` 屬性，將屬性系結至對應的事件處理常式。 例如，您可以使用下列兩個屬性，將 `MyProp` 的屬性系結至 `MyEventHandler`：

```cshtml
<MyComponent @bind-MyProp="MyValue" @bind-MyProp:event="MyEventHandler" />
```

## <a name="event-handling"></a>事件處理

Razor 元件提供事件處理功能。 針對名為 `on{event}` （例如，`onclick` 和 `onsubmit`）與委派類型值的 HTML 專案屬性，Razor 元件會將屬性的值視為事件處理常式。 屬性的名稱一律會格式化[@on {event}](xref:mvc/views/razor#onevent)。

下列程式碼會在 UI 中選取按鈕時，呼叫 `UpdateHeading` 方法：

```cshtml
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private void UpdateHeading(MouseEventArgs e)
    {
        ...
    }
}
```

下列程式碼會在 UI 中變更核取方塊時，呼叫 `CheckChanged` 方法：

```cshtml
<input type="checkbox" class="form-check-input" @onchange="CheckChanged" />

@code {
    private void CheckChanged()
    {
        ...
    }
}
```

事件處理常式也可以是非同步，並傳回 <xref:System.Threading.Tasks.Task>。 不需要手動呼叫 `StateHasChanged()`。 例外狀況會在發生時記錄。

在下列範例中，當選取按鈕時，會以非同步方式呼叫 `UpdateHeading`：

```cshtml
<button class="btn btn-primary" @onclick="UpdateHeading">
    Update heading
</button>

@code {
    private async Task UpdateHeading(MouseEventArgs e)
    {
        ...
    }
}
```

### <a name="event-argument-types"></a>事件引數類型

對於某些事件，則允許事件引數類型。 如果不需要存取這些事件種類的其中一個，則在方法呼叫中不需要。

支援的 `EventArgs` 會顯示在下表中。

| Event - 事件 | 執行個體 |
| ----- | ----- |
| 剪貼簿        | `ClipboardEventArgs` |
| 拖放式             | `DragEventArgs` &ndash; `DataTransfer` 和 `DataTransferItem` 會保存拖曳的專案資料。 |
| 錯誤            | `ErrorEventArgs` |
| 焦點            | `FocusEventArgs` &ndash; 不包含對 `relatedTarget` 的支援。 |
| `<input>` 變更 | `ChangeEventArgs` |
| 鍵盤         | `KeyboardEventArgs` |
| 滑鼠            | `MouseEventArgs` |
| 滑鼠指標    | `PointerEventArgs` |
| 滑鼠滾輪      | `WheelEventArgs` |
| 進度         | `ProgressEventArgs` |
| 觸控            | `TouchEventArgs` &ndash; `TouchPoint` 代表觸控裝置上的單一連絡人點。 |

如需上表中事件的屬性和事件處理行為的詳細資訊，請參閱[參考來源中的 EventArgs 類別（aspnet/AspNetCore release/3.0 分支）](https://github.com/aspnet/AspNetCore/tree/release/3.0/src/Components/Web/src/Web)。

### <a name="lambda-expressions"></a>Lambda 運算式

您也可以使用 Lambda 運算式：

```cshtml
<button @onclick="@(e => Console.WriteLine("Hello, world!"))">Say hello</button>
```

關閉其他值（例如反覆運算一組專案時）通常會很方便。 下列範例會建立三個按鈕，其中每個都會呼叫 `UpdateHeading` 會在 UI 中選取時傳遞事件引數（`MouseEventArgs`）和其按鈕編號（`buttonNumber`）：

```cshtml
<h2>@message</h2>

@for (var i = 1; i < 4; i++)
{
    var buttonNumber = i;

    <button class="btn btn-primary"
            @onclick="@(e => UpdateHeading(e, buttonNumber))">
        Button #@i
    </button>
}

@code {
    private string message = "Select a button to learn its position.";

    private void UpdateHeading(MouseEventArgs e, int buttonNumber)
    {
        message = $"You selected Button #{buttonNumber} at " +
            $"mouse position: {e.ClientX} X {e.ClientY}.";
    }
}
```

> [!NOTE]
> 請勿直接在 lambda 運算式**中使用 `for`** 迴圈中的迴圈變數（`i`）。 否則，所有 lambda 運算式都會使用相同的變數，使 `i` 值在所有 lambda 中都相同。 一律在本機變數中捕捉其值（在上述範例中為 `buttonNumber`），然後使用它。

### <a name="eventcallback"></a>App eventcallback

有一個常見的嵌套元件案例，就是當子元件事件發生時，想要執行父元件的方法 &mdash;for 例如，當子系中發生 `onclick` 事件時。 若要在元件之間公開事件，請使用 `EventCallback`。 父元件可以將回呼方法指派給子元件的 `EventCallback`。

範例應用程式中的 `ChildComponent` 會示範如何設定按鈕的 `onclick` 處理常式，以接收來自範例 `ParentComponent` 的 `EventCallback` 委派。 系統會使用 `MouseEventArgs` 輸入 `EventCallback`，這適用于來自週邊裝置的 `onclick` 事件：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=5-7,17-18)]

@No__t_0 會將子系的 `EventCallback<T>` 設定為其 `ShowMessage` 方法：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/ParentComponent.razor?name=snippet_ParentComponent&highlight=6,16-19)]

在 `ChildComponent` 中選取按鈕時：

* 呼叫 `ParentComponent` 的 `ShowMessage` 方法。 `messageText` 會更新，並顯示在 `ParentComponent` 中。
* 回呼的方法（`ShowMessage`）不需要呼叫 `StateHasChanged`。 `StateHasChanged` 會自動呼叫以 rerender `ParentComponent`，就像子事件會觸發元件 rerendering 在子系內執行的事件處理常式一樣。

`EventCallback` 和 `EventCallback<T>` 允許非同步委派。 `EventCallback<T>` 是強型別，而且需要特定的引數型別。 `EventCallback` 是弱式類型，且允許任何引數類型。

```cshtml
<p><b>@messageText</b></p>

@{ var message = "Default Text"; }

<ChildComponent 
    OnClick="@(async () => { await Task.Yield(); messageText = "Blaze It!"; })" />

@code {
    private string messageText;
}
```

使用 `InvokeAsync` 叫用 `EventCallback` 或 `EventCallback<T>`，並等待 <xref:System.Threading.Tasks.Task>：

```csharp
await callback.InvokeAsync(arg);
```

使用 `EventCallback`，並 `EventCallback<T>` 用於事件處理和系結元件參數。

偏好強型別 `EventCallback<T>`，而不是 `EventCallback`。 `EventCallback<T>` 會對元件的使用者提供更好的錯誤意見反應。 與其他 UI 事件處理常式類似，指定事件參數是選擇性的。 當沒有任何值傳遞至回呼時，請使用 `EventCallback`。

## <a name="chained-bind"></a>連鎖系結

常見的案例是將資料系結參數連結至元件輸出中的 page 元素。 此案例稱為*連鎖*系結，因為會同時發生多個層級的系結。

無法使用頁面的元素中的 `@bind` 語法來執行連鎖系結。 事件處理常式和值必須分別指定。 不過，父元件可以使用 `@bind` 語法搭配元件的參數。

下列 `PasswordField` 元件（*self.passwordfield.text*）：

* 將 `<input>` 元素的值設定為 `Password` 屬性。
* 將 `Password` 屬性的變更公開至具有[app eventcallback](#eventcallback)的父元件。

```cshtml
Password: 

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@Password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

@code {
    private bool showPassword;

    [Parameter]
    public string Password { get; set; }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        Password = e.Value.ToString();

        return PasswordChanged.InvokeAsync(Password);
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

@No__t_0 元件會在另一個元件中使用：

```cshtml
<PasswordField @bind-Password="password" />

@code {
    private string password;
}
```

若要對上述範例中的密碼執行檢查或陷印錯誤：

* 在下列範例程式碼中建立 `Password` 的支援欄位（`password`）。
* 執行 `Password` setter 中的檢查或陷阱錯誤。

如果密碼的值中使用了空格，下列範例會向使用者提供立即的意見反應：

```cshtml
Password: 

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@Password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

<span class="text-danger">@validationMessage</span>

@code {
    private bool showPassword;
    private string password;
    private string validationMessage;

    [Parameter]
    public string Password
    {
        get { return password ?? string.Empty; }
        set
        {
            if (password != value)
            {
                if (value.Contains(' '))
                {
                    validationMessage = "Spaces not allowed!";
                }
                else
                {
                    password = value;
                    validationMessage = string.Empty;
                }
            }
        }
    }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        Password = e.Value.ToString();

        return PasswordChanged.InvokeAsync(Password);
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

## <a name="capture-references-to-components"></a>捕獲元件的參考

元件參考提供參考元件實例的方法，讓您可以發出命令給該實例，例如 `Show` 或 `Reset`。 若要捕捉元件參考：

* 將[@ref](xref:mvc/views/razor#ref)屬性加入至子元件。
* 定義與子元件類型相同的欄位。

```cshtml
<MyLoginDialog @ref="loginDialog" ... />

@code {
    private MyLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

當元件轉譯時，[`loginDialog`] 欄位會填入 `MyLoginDialog` 子元件實例。 接著，您可以在元件實例上叫用 .NET 方法。

> [!IMPORTANT]
> 只有在轉譯元件且其輸出包含 `MyLoginDialog` 元素之後，才會填入 `loginDialog` 變數。 直到該點為止，沒有任何可參考的內容。 若要在元件完成呈現之後操作元件參考，請使用[OnAfterRenderAsync 或 OnAfterRender 方法](#lifecycle-methods)。

雖然捕捉元件參考使用類似的語法來[捕捉元素參考](xref:blazor/javascript-interop#capture-references-to-elements)，但它並不是[JavaScript interop](xref:blazor/javascript-interop)功能。 元件參考不會傳遞至 JavaScript 程式碼，&mdash;they 只會在 .NET 程式碼中使用。

> [!NOTE]
> 請勿**使用元件**參考來改變子元件的狀態。 請改用一般宣告式參數，將資料傳遞至子元件。 使用一般宣告式參數會導致子元件自動 rerender 正確的時間。

## <a name="invoke-component-methods-externally-to-update-state"></a>在外部叫用元件方法來更新狀態

Blazor 會使用 `SynchronizationContext` 來強制執行單一邏輯執行緒。 元件的生命週期方法以及 Blazor 所引發的任何事件回呼都會在此 `SynchronizationContext` 上執行。 在事件中，必須根據外來事件（例如計時器或其他通知）更新元件，請使用 `InvokeAsync` 方法，這會分派給 Blazor 的 `SynchronizationContext`。

例如，假設有一個通知程式*服務*可通知任何處于已更新狀態的「接聽」元件：

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

使用 `NotifierService` 來更新元件：

```cshtml
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

在上述範例中，`NotifierService` 會叫用元件在 Blazor 的 `SynchronizationContext` 以外的 `OnNotify` 方法。 `InvokeAsync` 是用來切換至正確的內容，並將轉譯排入佇列。

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>使用 \@key 控制元素和元件的保留

當轉譯專案或元件的清單，以及後續變更的專案或元件時，Blazor 的比較演算法必須決定哪些先前的專案或元件可以保留，以及模型物件應如何對應至這些專案。 一般來說，此程式是自動的，可以忽略，但在某些情況下，您可能會想要控制進程。

參考下列範例：

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

@No__t_0 集合的內容可能會隨著插入、刪除或重新排序的專案而變更。 當元件 rerenders 時，`<DetailsEditor>` 元件可能會變更，以接收不同的 `Details` 參數值。 這可能會導致比預期更複雜的 rerendering。 在某些情況下，rerendering 可能會導致可見的行為差異，例如失去元素的焦點。

您可以使用 `@key` 指示詞屬性來控制對應進程。 `@key` 會使比較演算法根據索引鍵的值，保證保留元素或元件：

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

當 `People` 集合變更時，比較演算法會保留 `<DetailsEditor>` 實例與 `person` 實例之間的關聯：

* 如果從 `People` 清單中刪除 `Person`，則只會從 UI 移除對應的 `<DetailsEditor>` 實例。 其他實例則保持不變。
* 如果在清單中的某個位置插入 `Person`，就會在該對應的位置插入一個新的 `<DetailsEditor>` 實例。 其他實例則保持不變。
* 如果重新排序 `Person` 專案，則會保留對應的 `<DetailsEditor>` 實例，並在 UI 中重新排序。

在某些情況下，使用 `@key` 會將 rerendering 的複雜性降到最低，並避免 DOM 的具狀態部分可能發生的問題，例如焦點位置。

> [!IMPORTANT]
> 索引鍵在每個容器元素或元件的本機。 金鑰不會在檔之間進行全域比較。

### <a name="when-to-use-key"></a>使用 \@key 的時機

一般而言，只要轉譯清單（例如，在 `@foreach` 區塊中），而且有適當的值來定義 `@key`，就有合理的使用 `@key`。

當物件變更時，您也可以使用 `@key` 來防止 Blazor 保留元素或元件子樹：

```cshtml
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

如果 `@currentPerson` 變更，`@key` attribute 指示詞會強制 Blazor 捨棄整個 `<div>` 及其下階，並使用新的元素和元件重建 UI 內的子樹。 如果您需要保證 `@currentPerson` 變更時不會保留任何 UI 狀態，這會很有用。

### <a name="when-not-to-use-key"></a>當不使用時 \@key

與 `@key` 進行比較時，會產生效能成本。 效能成本並不大，但只有在控制元素或元件保留規則以受益于應用程式時，才指定 `@key`。

即使未使用 `@key`，Blazor 還是會盡可能保留子項目和元件實例。 使用 `@key` 的唯一優點是控制模型實例*如何*對應至保留的元件實例，而不是用來選取對應的比較演算法。

### <a name="what-values-to-use-for-key"></a>要用於 \@key 的值

一般來說，提供下列其中一種類型的值給 `@key` 是合理的：

* 模型物件實例（例如，如先前範例所示的 `Person` 實例）。 這可確保根據物件參考的相等性進行保留。
* 唯一識別碼（例如，類型 `int`、`string` 或 `Guid`）的主要金鑰值。

請確定用於 `@key` 的值不會造成衝突。 如果在相同的父元素中偵測到衝突值，Blazor 會擲回例外狀況，因為它無法以決定性的方式將舊專案或元件對應到新的專案或元件。 只使用不同的值，例如物件實例或主鍵值。

## <a name="lifecycle-methods"></a>生命週期方法

`OnInitializedAsync` 和 `OnInitialized` 會執行程式碼以初始化元件。 若要執行非同步作業，請在作業上使用 `OnInitializedAsync` 和 `await` 關鍵字：

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

> [!NOTE]
> 在 `OnInitializedAsync` 週期事件期間，必須在元件初始化期間進行非同步工作。

如需同步操作，請使用 `OnInitialized`：

```csharp
protected override void OnInitialized()
{
    ...
}
```

當元件已從其父系接收參數，且已將值指派給屬性時，會呼叫 `OnParametersSetAsync` 和 `OnParametersSet`。 這些方法會在元件初始化之後和每次呈現元件時執行：

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> 當套用參數和屬性值時，非同步工作必須在 `OnParametersSetAsync` 週期事件期間發生。

```csharp
protected override void OnParametersSet()
{
    ...
}
```

在元件完成呈現之後，會呼叫 `OnAfterRenderAsync` 和 `OnAfterRender`。 此時會填入元素和元件參考。 使用此階段來執行使用轉譯內容的其他初始化步驟，例如啟用在轉譯的 DOM 元素上操作的協力廠商 JavaScript 程式庫。

在*伺服器上預先呈現時，不會呼叫*`OnAfterRender`。

@No__t_1 和 `OnAfterRender` 的 `firstRender` 參數是：

* 當第一次叫用元件實例時，設定為 `true`。
* 確保初始化工作只會執行一次。

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> 在 `OnAfterRenderAsync` 週期事件期間，必須立即執行非同步工作。

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

### <a name="handle-incomplete-async-actions-at-render"></a>處理轉譯時的未完成非同步動作

在呈現元件之前，在生命週期事件中執行的非同步動作可能尚未完成。 當生命週期方法正在執行時，物件可能 `null` 或未完全填入資料。 提供轉譯邏輯，以確認物件已初始化。 當物件 `null` 時，轉譯預留位置 UI 元素（例如，載入訊息）。

在 Blazor 範本的 `FetchData` 元件中，`OnInitializedAsync` 會覆寫為 asychronously 接收預測資料（`forecasts`）。 當 `forecasts` `null` 時，會向使用者顯示載入訊息。 @No__t_1 所傳回的 `Task` 完成之後，元件會以更新的狀態重新顯示。

*Pages/FetchData.razor*：

[!code-cshtml[](components/samples_snapshot/3.x/FetchData.razor?highlight=9)]

### <a name="execute-code-before-parameters-are-set"></a>在設定參數之前執行程式碼

在設定參數之前，可以覆寫 `SetParameters` 來執行程式碼：

```csharp
public override void SetParameters(ParameterView parameters)
{
    ...

    base.SetParameters(parameters);
}
```

如果未叫用 `base.SetParameters`，自訂程式碼就可以任何需要的方式解讀傳入的參數值。 例如，傳入的參數不需要指派給類別的屬性。

### <a name="suppress-refreshing-of-the-ui"></a>隱藏 UI 的重新整理

可以覆寫 `ShouldRender`，以隱藏 UI 的重新整理。 如果執行傳回 `true`，則會重新整理 UI。 即使 `ShouldRender` 遭到覆寫，元件一律會一開始呈現。

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

## <a name="component-disposal-with-idisposable"></a>使用 IDisposable 的元件處置

如果元件執行 <xref:System.IDisposable>，則會在從 UI 移除元件時呼叫[Dispose 方法](/dotnet/standard/garbage-collection/implementing-dispose)。 下列元件會使用 `@implements IDisposable` 和 `Dispose` 方法：

```csharp
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

> [!NOTE]
> 不支援在 `Dispose` 中呼叫 `StateHasChanged`。 `StateHasChanged` 可能會在轉譯器關閉時叫用。 在該時間點不支援要求 UI 更新。

## <a name="routing"></a>路由

Blazor 中的路由是藉由將路由範本提供給應用程式中每個可存取的元件來達成。

當您編譯具有 `@page` 指示詞的 Razor 檔案時，會提供指定路由範本 <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> 給產生的類別。 在執行時間，路由器會尋找具有 `RouteAttribute` 的元件類別，並轉譯哪一個元件的路由範本符合要求的 URL。

多個路由範本可以套用至元件。 下列元件會回應 `/BlazorRoute` 和 `/DifferentBlazorRoute` 的要求：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/BlazorRoute.razor?name=snippet_BlazorRoute)]

## <a name="route-parameters"></a>路由參數

元件可以從 `@page` 指示詞中提供的路由範本接收路由參數。 路由器會使用路由參數來填入對應的元件參數。

*路由參數元件*：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/RouteParameter.razor?name=snippet_RouteParameter)]

不支援選擇性參數，因此上述範例中會套用兩個 `@page` 的指示詞。 第一個則允許不使用參數導覽至元件。 第二個 `@page` 指示詞會採用 `{text}` 路由參數，並將值指派給 `Text` 屬性。

::: moniker range=">= aspnetcore-3.1"

## <a name="partial-class-support"></a>部分類別支援

Razor 元件是以部分類別的形式產生。 Razor 元件是使用下列其中一種方法來撰寫的：

* C#程式碼是以 HTML 標籤和 Razor 程式碼的[@code](xref:mvc/views/razor#code)區塊定義在單一檔案中。 Blazor 範本會使用這種方法來定義其 Razor 元件。
* C#程式碼會放在定義為部分類別的程式碼後置檔案中。

下列範例顯示在 Blazor 範本所產生的應用程式中，具有 `@code` 區塊的預設 `Counter` 元件。 HTML 標籤、Razor 程式碼和C#程式碼位於相同的檔案中：

*Counter. razor*：

```cshtml
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

您也可以使用具有部分類別的程式碼後置檔案來建立 `Counter` 元件：

*Counter. razor*：

```cshtml
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
```

*Counter.razor.cs*：

```csharp
namespace BlazorApp.Pages
{
    public partial class Counter
    {
        int currentCount = 0;

        void IncrementCount()
        {
            currentCount++;
        }
    }
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.1"

## <a name="specify-a-component-base-class"></a>指定元件基類

@No__t_0 指示詞可以用來指定元件的基類。

[範例應用程式](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)會顯示元件如何繼承基類（`BlazorRocksBase`），以提供元件的屬性和方法。

*Pages/BlazorRocks. razor*：

```cshtml
@page "/BlazorRocks"
@inherits BlazorRocksBase

<h1>@BlazorRocksText</h1>
```

*BlazorRocksBase.cs*：

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample
{
    public class BlazorRocksBase : ComponentBase
    {
        public string BlazorRocksText { get; set; } = 
            "Blazor rocks the browser!";
    }
}
```

基類應該衍生自 `ComponentBase`。

::: moniker-end

## <a name="import-components"></a>匯入元件

以 Razor 撰寫之元件的命名空間是根據（依優先順序排列）：

* Razor file （*razor*）標記中[的 @namespace](xref:mvc/views/razor#namespace)指定（`@namespace BlazorSample.MyNamespace`）。
* 專案檔中的專案 `RootNamespace` （`<RootNamespace>BlazorSample</RootNamespace>`）。
* 從專案檔的檔案名（ *.csproj*）取得的專案名稱，以及從專案根目錄到元件的路徑。 例如，架構會將 *{PROJECT ROOT}/Pages/Index.razor* （*BlazorSample*）解析為命名空間 `BlazorSample.Pages`。 元件會C#遵循名稱系結規則。 針對此範例中的 `Index` 元件，範圍內的元件是所有元件：
  * 在相同的資料夾中，*頁面*。
  * 專案根目錄中未明確指定不同命名空間的元件。

使用 Razor 的[@using](xref:mvc/views/razor#using)指示詞，在不同的命名空間中定義的元件會帶入範圍中。

如果*BlazorSample/Shared/* 資料夾中有另一個元件（`NavMenu.razor`）存在，則元件可用於 `Index.razor`，並具有下列 `@using` 個語句：

```cshtml
@using BlazorSample.Shared

This is the Index page.

<NavMenu></NavMenu>
```

元件也可以使用其完整名稱來參考，這不需要[@using](xref:mvc/views/razor#using)指示詞：

```cshtml
This is the Index page.

<BlazorSample.Shared.NavMenu></BlazorSample.Shared.NavMenu>
```

> [!NOTE]
> 不支援 `global::` 限定。
>
> 不支援以別名 `using` 的語句匯入元件（例如，`@using Foo = Bar`）。
>
> 不支援部分限定的名稱。 例如，不支援在 `<Shared.NavMenu></Shared.NavMenu>` 中加入 `@using BlazorSample` 和參考 `NavMenu.razor`。

## <a name="conditional-html-element-attributes"></a>條件式 HTML 元素屬性

HTML 專案屬性會根據 .NET 值有條件地呈現。 如果值為 `false` 或 `null`，則不會呈現屬性。 如果值是 `true`，則會將屬性轉譯為最小化。

在下列範例中，`IsCompleted` 會決定是否要在專案的標記中呈現 `checked`：

```cshtml
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

如果 `IsCompleted` `true`，則會將此核取方塊轉譯為：

```html
<input type="checkbox" checked />
```

如果 `IsCompleted` `false`，則會將此核取方塊轉譯為：

```html
<input type="checkbox" />
```

如需詳細資訊，請參閱<xref:mvc/views/razor>。

> [!WARNING]
> 當 .NET 類型為 `bool` 時，某些 HTML 屬性（例如，由[aria 按下](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons)）無法正常運作。 在這些情況下，請使用 `string` 類型，而不是 `bool`。

## <a name="raw-html"></a>原始 HTML

字串通常會使用 DOM 文位元組點來呈現，這表示它們可能包含的任何標記都會被忽略，並視為常值。 若要呈現原始 HTML，請將 HTML 內容包裝 `MarkupString` 值。 此值會剖析為 HTML 或 SVG，並插入 DOM 中。

> [!WARNING]
> 轉譯從任何未受信任來源所建立的原始 HTML 會有**安全性風險**，應予以避免！

下列範例顯示如何使用 `MarkupString` 型別，將靜態 HTML 內容的區塊新增至元件的轉譯輸出：

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="templated-components"></a>樣板化元件

樣板化元件是接受一或多個 UI 範本做為參數的元件，然後可以用來做為元件轉譯邏輯的一部分。 樣板化元件可讓您撰寫比一般元件更容易重複使用的較高層級元件。 其中有幾個範例包括：

* 資料表元件，可讓使用者指定資料表標頭、資料列和頁尾的範本。
* 清單元件，可讓使用者指定範本來轉譯清單中的專案。

### <a name="template-parameters"></a>範本參數

樣板化元件是藉由指定一或多個類型的元件參數 `RenderFragment` 或 `RenderFragment<T>` 來定義。 呈現片段代表要呈現的 UI 區段。 `RenderFragment<T>` 會採用可在叫用轉譯片段時指定的類型參數。

`TableTemplate` 元件：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/TableTemplate.razor)]

使用樣板化元件時，可以使用符合參數名稱的子專案來指定範本參數（`TableHeader`，並在下列範例中 `RowTemplate`）：

```cshtml
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@context.PetId</td>
        <td>@context.Name</td>
    </RowTemplate>
</TableTemplate>
```

### <a name="template-context-parameters"></a>範本內容參數

當做元素傳遞之類型 `RenderFragment<T>` 的元件引數具有名為 `context` 的隱含參數（例如，從前面的程式碼範例，`@context.PetId`），但您可以使用子專案上的 `Context` 屬性來變更參數名稱。 在下列範例中，`RowTemplate` 元素的 `Context` 屬性會指定 `pet` 參數：

```cshtml
<TableTemplate Items="pets">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate Context="pet">
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

或者，您也可以在 component 元素上指定 `Context` 屬性。 指定的 `Context` 屬性會套用至所有指定的範本參數。 當您想要指定隱含子內容的內容參數名稱時（不含任何換行的子項目），這會很有用。 在下列範例中，`Context` 屬性會出現在 `TableTemplate` 元素上，並套用至所有範本參數：

```cshtml
<TableTemplate Items="pets" Context="pet">
    <TableHeader>
        <th>ID</th>
        <th>Name</th>
    </TableHeader>
    <RowTemplate>
        <td>@pet.PetId</td>
        <td>@pet.Name</td>
    </RowTemplate>
</TableTemplate>
```

### <a name="generic-typed-components"></a>泛型型別元件

樣板化元件通常會以一般方式輸入。 例如，泛型 `ListViewTemplate` 元件可以用來呈現 `IEnumerable<T>` 值。 若要定義泛型元件，請使用[@typeparam](xref:mvc/views/razor#typeparam)指示詞來指定類型參數：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/ListViewTemplate.razor)]

使用泛型型別元件時，會在可能的情況下推斷型別參數：

```cshtml
<ListViewTemplate Items="pets">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

否則，必須使用符合型別參數名稱的屬性來明確指定型別參數。 在下列範例中，`TItem="Pet"` 會指定類型：

```cshtml
<ListViewTemplate Items="pets" TItem="Pet">
    <ItemTemplate Context="pet">
        <li>@pet.Name</li>
    </ItemTemplate>
</ListViewTemplate>
```

## <a name="cascading-values-and-parameters"></a>級聯的值和參數

在某些情況下，使用[元件參數](#component-parameters)將資料從上階元件傳送到子元件是不方便的，特別是在有數個元件層時。 串聯的值和參數可讓上階元件提供一個值給其所有子系元件，藉此解決這個問題。 級聯的值和參數也會提供一種方法來協調元件。

### <a name="theme-example"></a>主題範例

在範例應用程式的下列範例中，`ThemeInfo` 類別會指定要向下流動元件階層的主題資訊，讓應用程式中指定部分內的所有按鈕共用相同的樣式。

*UIThemeClasses/ThemeInfo .cs*：

```csharp
public class ThemeInfo
{
    public string ButtonClass { get; set; }
}
```

祖系元件可以使用串聯值元件來提供串聯值。 @No__t_0 元件會包裝元件階層的子樹，並提供單一值給該子樹內的所有元件。

例如，範例應用程式會在其中一個應用程式的配置中，將主題資訊（`ThemeInfo`）指定為構成 `@Body` 屬性之配置主體的所有元件的串聯參數。 `ButtonClass` 會在版面配置元件中指派 `btn-success` 的值。 任何子代元件都可以透過 `ThemeInfo` 級聯的物件來取用這個屬性。

`CascadingValuesParametersLayout` 元件：

```cshtml
@inherits LayoutComponentBase
@using BlazorSample.UIThemeClasses

<div class="container-fluid">
    <div class="row">
        <div class="col-sm-3">
            <NavMenu />
        </div>
        <div class="col-sm-9">
            <CascadingValue Value="theme">
                <div class="content px-4">
                    @Body
                </div>
            </CascadingValue>
        </div>
    </div>
</div>

@code {
    private ThemeInfo theme = new ThemeInfo { ButtonClass = "btn-success" };
}
```

為了利用串聯值，元件會使用 `[CascadingParameter]` 屬性宣告串聯式參數。 串聯式值會依類型系結至串聯式參數。

在範例應用程式中，`CascadingValuesParametersTheme` 元件會將 `ThemeInfo` 級聯的值系結至串聯式參數。 參數是用來為元件所顯示的其中一個按鈕設定 CSS 類別。

`CascadingValuesParametersTheme` 元件：

```cshtml
@page "/cascadingvaluesparameterstheme"
@layout CascadingValuesParametersLayout
@using BlazorSample.UIThemeClasses

<h1>Cascading Values & Parameters</h1>

<p>Current count: @currentCount</p>

<p>
    <button class="btn" @onclick="IncrementCount">
        Increment Counter (Unthemed)
    </button>
</p>

<p>
    <button class="btn @ThemeInfo.ButtonClass" @onclick="IncrementCount">
        Increment Counter (Themed)
    </button>
</p>

@code {
    private int currentCount = 0;

    [CascadingParameter]
    protected ThemeInfo ThemeInfo { get; set; }

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

若要在相同的子樹中串聯多個相同類型的值，請為每個 `CascadingValue` 元件和其對應的 `CascadingParameter` 提供唯一的 `Name` 字串。 在下列範例中，兩個 `CascadingValue` 元件會依名稱將不同的實例 `MyCascadingType`：

```cshtml
<CascadingValue Value=@ParentCascadeParameter1 Name="CascadeParam1">
    <CascadingValue Value=@ParentCascadeParameter2 Name="CascadeParam2">
        ...
    </CascadingValue>
</CascadingValue>

@code {
    private MyCascadingType ParentCascadeParameter1;

    [Parameter]
    public MyCascadingType ParentCascadeParameter2 { get; set; }

    ...
}
```

在子系元件中，串聯的參數會以名稱從上階元件中對應的串聯值接收其值：

```cshtml
...

@code {
    [CascadingParameter(Name = "CascadeParam1")]
    protected MyCascadingType ChildCascadeParameter1 { get; set; }
    
    [CascadingParameter(Name = "CascadeParam2")]
    protected MyCascadingType ChildCascadeParameter2 { get; set; }
}
```

### <a name="tabset-example"></a>TabSet 範例

串聯式參數也可以讓元件在元件階層之間共同作業。 例如，請考慮範例應用程式中的下列*TabSet*範例。

範例應用程式有一個 `ITab` 的介面，可執行索引標籤：

[!code-csharp[](common/samples/3.x/BlazorWebAssemblySample/UIInterfaces/ITab.cs)]

@No__t_0 元件會使用 `TabSet` 元件，其中包含數個 `Tab` 元件：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Pages/CascadingValuesParametersTabSet.razor?name=snippet_TabSet)]

子 `Tab` 元件並未明確地當做參數傳遞至 `TabSet`。 相反地，子 `Tab` 元件是 `TabSet` 之子內容的一部分。 不過，`TabSet` 還是必須知道每個 `Tab` 元件，才能轉譯標頭和使用中的索引標籤。若要在不需要額外程式碼的情況下啟用這項協調，`TabSet` 元件*可以將其本身提供為*串聯的值，然後由子 `Tab` 元件挑選。

`TabSet` 元件：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/TabSet.razor)]

子系 `Tab` 元件會將包含的 `TabSet` 作為串聯參數來捕捉，因此 `Tab` 元件會將自己加入至 `TabSet`，並對作用中的索引標籤進行協調。

`Tab` 元件：

[!code-cshtml[](common/samples/3.x/BlazorWebAssemblySample/Components/Tab.razor)]

## <a name="razor-templates"></a>Razor 範本

轉譯片段可以使用 Razor 範本語法來定義。 Razor 範本是定義 UI 程式碼片段並採用下列格式的方式：

```cshtml
@<{HTML tag}>...</{HTML tag}>
```

下列範例說明如何指定 `RenderFragment` 和 `RenderFragment<T>` 值，並直接在元件中呈現範本。 轉譯片段也可以當做引數傳遞至樣板[化元件](#templated-components)。

```cshtml
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = 
        (pet) => @<p>Your pet's name is @pet.Name.</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

上述程式碼的轉譯輸出：

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Your pet's name is Rex.</p>
```

## <a name="manual-rendertreebuilder-logic"></a>手動 RenderTreeBuilder 邏輯

`Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder` 提供操作元件和元素的方法，包括在程式碼中C#手動建立元件。

> [!NOTE]
> 使用 `RenderTreeBuilder` 來建立元件是一個先進的案例。 格式不正確的元件（例如，未封閉的標記標記）可能會導致未定義的行為。

請考慮下列 `PetDetails` 元件，其可手動內建于另一個元件中：

```cshtml
<h2>Pet Details Component</h2>

<p>@PetDetailsQuote</p>

@code
{
    [Parameter]
    public string PetDetailsQuote { get; set; }
}
```

在下列範例中，`CreateComponent` 方法中的迴圈會產生三個 `PetDetails` 元件。 呼叫 `RenderTreeBuilder` 方法來建立元件（`OpenComponent` 和 `AddAttribute`）時，序號是源程式碼號。 Blazor 差異演算法依賴對應于不同程式程式碼的序號，而不是相異的呼叫調用。 建立具有 `RenderTreeBuilder` 方法的元件時，請硬式編碼序號的引數。 **使用計算或計數器來產生序號可能會導致效能不佳。** 如需詳細資訊，請參閱[序號與程式程式碼號，而不是執行順序](#sequence-numbers-relate-to-code-line-numbers-and-not-execution-order)一節。

`BuiltContent` 元件：

```cshtml
@page "/BuiltContent"

<h1>Build a component</h1>

@CustomRender

<button type="button" @onclick="RenderComponent">
    Create three Pet Details components
</button>

@code {
    private RenderFragment CustomRender { get; set; }
    
    private RenderFragment CreateComponent() => builder =>
    {
        for (var i = 0; i < 3; i++) 
        {
            builder.OpenComponent(0, typeof(PetDetails));
            builder.AddAttribute(1, "PetDetailsQuote", "Someone's best friend!");
            builder.CloseComponent();
        }
    };    
    
    private void RenderComponent()
    {
        CustomRender = CreateComponent();
    }
}
```

> !WARNING@No__t_0 中的類型允許處理轉譯作業的*結果*。 這些是 Blazor framework 執行的內部詳細資料。 這些類型應該被視為不*穩定*，未來的版本可能會變更。

### <a name="sequence-numbers-relate-to-code-line-numbers-and-not-execution-order"></a>序號與程式程式碼號相關，而不是執行順序

Blazor `.razor`：一律會編譯檔案。 這可能是 `.razor` 的絕佳優點，因為您可以使用編譯步驟來插入資訊，以在執行時間改善應用程式效能。

這些改良功能的重要範例包括*序號*。 序號會向運行時程表示輸出來自哪些不同和已排序的程式程式碼。 執行時間會使用這項資訊，以線性時間產生有效率的樹狀差異，這比一般樹狀結構的差異演算法通常還能快得多。

請考慮下列簡單的 `.razor` 檔案：

```cshtml
@if (someFlag)
{
    <text>First</text>
}

Second
```

上述程式碼會編譯成如下所示的內容：

```csharp
if (someFlag)
{
    builder.AddContent(0, "First");
}

builder.AddContent(1, "Second");
```

當程式碼第一次執行時，如果 `someFlag` 是 `true`，產生器會接收：

| 序列 | 輸入      | 資料   |
| :------: | --------- | :----: |
| 0        | Text node | First  |
| 1        | Text node | Second |

假設 `someFlag` 會變成 `false`，並再次轉譯標記。 這次，產生器會接收：

| 序列 | 輸入       | 資料   |
| :------: | ---------- | :----: |
| 1        | Text node  | Second |

當執行時間執行差異時，會看到順序 `0` 的專案已移除，因此它會產生下列簡單的*編輯腳本*：

* 移除第一個文位元組點。

#### <a name="what-goes-wrong-if-you-generate-sequence-numbers-programmatically"></a>當您以程式設計方式產生序號時，會發生什麼錯誤

想像一下，您會改為撰寫下列轉譯樹產生器邏輯：

```csharp
var seq = 0;

if (someFlag)
{
    builder.AddContent(seq++, "First");
}

builder.AddContent(seq++, "Second");
```

現在，第一個輸出是：

| 序列 | 輸入      | 資料   |
| :------: | --------- | :----: |
| 0        | Text node | First  |
| 1        | Text node | Second |

此結果與先前的案例相同，因此不會有負面問題存在。 `someFlag` 在第二個轉譯中是 `false`，輸出則是：

| 序列 | 輸入      | 資料   |
| :------: | --------- | ------ |
| 0        | Text node | Second |

這次，diff 演算法發現發生了*兩*項變更，而演算法會產生下列編輯腳本：

* 將第一個文位元組點的值變更為 `Second`。
* 移除第二個文位元組點。

產生序號已遺失有關 `if/else` 分支和迴圈出現在原始程式碼中的所有實用資訊。 這會導致差異**兩倍，但前提**是之前。

這是一個簡單的範例。 在具有複雜和深層嵌套結構的更真實案例中，尤其是使用迴圈時，效能成本會更嚴重。 Diff 演算法不會立即識別已插入或移除的迴圈區塊或分支，而是必須在轉譯樹狀結構中進行深入的遞迴，而且通常會建立更長的編輯腳本，因為它 misinformed 了新舊結構的方式。相互關聯。

#### <a name="guidance-and-conclusions"></a>指引和結論

* 如果序號是動態產生的，應用程式效能會受到影響。
* 架構無法在執行時間自動建立自己的序號，因為必要的資訊不存在，除非是在編譯時期加以捕捉。
* 請勿撰寫以手動方式執行的長區塊，`RenderTreeBuilder` 邏輯。 偏好 `.razor` 檔案，並允許編譯器處理序號。 如果您無法避免手動 `RenderTreeBuilder` 邏輯，請將長塊的程式碼分割成較小的片段，並以 `OpenRegion` / `CloseRegion` 呼叫來包裝。 每個區域都有自己的序號個別空間，因此您可以在每個區域內從零（或任何其他任一數字）重新開機。
* 如果序號已硬式編碼，則 diff 演算法只會要求序號增加值。 起始值和間距無關。 一個合法的選項是使用程式程式碼號做為序號，或從零開始，並以一個或數百個（或任何慣用的間隔）來增加。 
* Blazor 會使用序號，而其他樹狀結構比較的 UI 架構則不會使用它們。 使用序號時，比較速度會更快，而且 Blazor 具有可自動處理序號的編譯步驟，讓開發人員撰寫 `.razor` 檔案。

## <a name="localization"></a>當地語系化

Blazor 伺服器應用程式是使用[當地語系化中介軟體](xref:fundamentals/localization#localization-middleware)來當地語系化。 中介軟體會為從應用程式要求資源的使用者選取適當的文化特性。

您可以使用下列其中一種方法來設定文化特性：

* [Cookie](#cookies)
* [提供 UI 以選擇文化特性](#provide-ui-to-choose-the-culture)

如需詳細資訊與範例，請參閱<xref:fundamentals/localization>。

### <a name="cookies"></a>Cookie

當地語系化文化特性 cookie 可以保存使用者的文化特性。 Cookie 是由應用程式主機頁面的 `OnGet` 方法所建立（*Pages/主機. cshtml .cs*）。 當地語系化中介軟體會在後續要求中讀取 cookie，以設定使用者的文化特性。 

使用 cookie 可確保 WebSocket 連接可以正確地傳播文化特性。 如果當地語系化架構是以 URL 路徑或查詢字串為基礎，配置可能無法使用 Websocket，因而無法保存文化特性。 因此，建議的方法是使用當地語系化文化特性 cookie。

如果文化特性保存在當地語系化 cookie 中，任何技術都可以用來指派文化特性。 如果應用程式已建立伺服器端 ASP.NET Core 的當地語系化配置，請繼續使用應用程式的現有當地語系化基礎結構，並在應用程式的配置內設定當地語系化文化特性 cookie。

下列範例顯示如何在可由當地語系化中介軟體讀取的 cookie 中設定目前的文化特性。 在 Blazor 伺服器應用程式中，使用下列內容建立*Pages/主機. cshtml*檔案：

```csharp
public class HostModel : PageModel
{
    public void OnGet()
    {
        HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }
}
```

當地語系化會在應用程式中處理：

1. 瀏覽器會將初始 HTTP 要求傳送至應用程式。
1. 文化特性是由當地語系化中介軟體所指派。
1. *_Host*中的 `OnGet` 方法會在 cookie 中保存文化特性，做為回應的一部分。
1. 瀏覽器會開啟 WebSocket 連線，以建立互動式 Blazor 伺服器會話。
1. 當地語系化中介軟體會讀取 cookie 並指派文化特性。
1. Blazor 伺服器會話的開頭是正確的文化特性。

## <a name="provide-ui-to-choose-the-culture"></a>提供 UI 以選擇文化特性

為了提供允許使用者選取文化特性的 UI，建議使用以重新*導向為基礎的方法*。 此程式類似于在使用者嘗試存取安全資源時，在 web 應用程式中發生的情況，&mdash;the 使用者重新導向至登入頁面，然後重新導向至原始資源。 

應用程式會透過重新導向至控制器的方式，來保存使用者選取的文化特性。 控制器會將使用者選取的文化特性設定為 cookie，並將使用者重新導向回到原始 URI。

在伺服器上建立 HTTP 端點，以在 cookie 中設定使用者選取的文化特性，並執行重新導向回到原始 URI：

```csharp
[Route("[controller]/[action]")]
public class CultureController : Controller
{
    public IActionResult SetCulture(string culture, string redirectUri)
    {
        if (culture != null)
        {
            HttpContext.Response.Cookies.Append(
                CookieRequestCultureProvider.DefaultCookieName,
                CookieRequestCultureProvider.MakeCookieValue(
                    new RequestCulture(culture)));
        }

        return LocalRedirect(redirectUri);
    }
}
```

> [!WARNING]
> 使用 [`LocalRedirect`] 動作結果來防止開啟重新導向攻擊。 如需詳細資訊，請參閱<xref:security/preventing-open-redirects>。

下列元件顯示當使用者選取文化特性時，如何執行初始重新導向的範例：

```cshtml
@inject NavigationManager NavigationManager

<h3>Select your language</h3>

<select @onchange="OnSelected">
    <option>Select...</option>
    <option value="en-US">English</option>
    <option value="fr-FR">Français</option>
</select>

@code {
    private double textNumber;

    private void OnSelected(ChangeEventArgs e)
    {
        var culture = (string)e.Value;
        var uri = new Uri(NavigationManager.Uri())
            .GetComponents(UriComponents.PathAndQuery, UriFormat.Unescaped);
        var query = $"?culture={Uri.EscapeDataString(culture)}&" +
            $"redirectUri={Uri.EscapeDataString(uri)}";

        NavigationManager.NavigateTo("/Culture/SetCulture" + query, forceLoad: true);
    }
}
```

### <a name="use-net-localization-scenarios-in-blazor-apps"></a>在 Blazor apps 中使用 .NET 當地語系化案例

在 Blazor apps 中，可以使用下列 .NET 當地語系化和全球化案例：

* .NET 的資源系統
* 特定文化特性的數位和日期格式

Blazor 的 `@bind` 功能會根據使用者目前的文化特性來執行全球化。 如需詳細資訊，請參閱[資料](#data-binding)系結一節。

目前支援一組有限的 ASP.NET Core 當地語系化案例：

* Blazor apps*支援*`IStringLocalizer<>`。
* `IHtmlLocalizer<>`、`IViewLocalizer<>` 和資料批註當地語系化會 ASP.NET Core MVC 案例中，而且**不支援**Blazor 應用程式。

如需詳細資訊，請參閱<xref:fundamentals/localization>。

## <a name="scalable-vector-graphics-svg-images"></a>可擴充向量圖形（SVG）影像

由於 Blazor 會轉譯 HTML，瀏覽器支援的影像（包括可擴充的向量圖形（SVG）影像（*svg*））可透過 `<img>` 標記來支援：

```html
<img alt="Example image" src="some-image.svg" />
```

同樣地，樣式表單檔案（ *.css*）的 CSS 規則也支援 SVG 影像：

```css
.my-element {
    background-image: url("some-image.svg");
}
```

不過，在所有案例中不支援內嵌 SVG 標記。 如果您將 `<svg>` 標記直接放入元件檔案（*razor*），則會支援基本映射轉譯，但尚不支援許多先進的案例。 例如，目前未遵守 `<use>` 標籤，且 `@bind` 無法與某些 SVG 標記搭配使用。 我們希望在未來的版本中解決這些限制。

## <a name="additional-resources"></a>其他資源

* <xref:security/blazor/server> &ndash; 包含有關建立 Blazor 伺服器應用程式的指導方針，必須對抗資源耗盡。

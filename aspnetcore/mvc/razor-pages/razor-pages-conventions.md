---
title: 'Razor-Seiten: Routen- und App-Konventionen in ASP.NET Core'
author: guardrex
description: Erfahren Sie, wie Konventionen für Routen- und App-Modellanbieter Sie beim Steuern von Seitenrouting, Ermittlung und Verarbeitung unterstützen können.
manager: wpickett
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.date: 04/12/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/razor-pages/razor-pages-conventions
ms.openlocfilehash: eba3422fbf46ac181a783b7f8cc605c2a549b4b7
ms.sourcegitcommit: 43bd79667bbdc8a07bd39fb4cd6f7ad3e70212fb
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34729742"
---
# <a name="razor-pages-route-and-app-conventions-in-aspnet-core"></a>Razor-Seiten: Routen- und App-Konventionen in ASP.NET Core

Von [Luke Latham](https://github.com/guardrex)

Erfahren Sie, wie Sie in Apps für Razor-Seiten mithilfe von [Konventionen für Seitenrouten und App-Modellanbieter](xref:mvc/controllers/application-model#conventions) das Seitenrouting, die Ermittlung und die Verarbeitung steuern können. Wenn Sie für einzelne Seiten benutzerdefinierte Seitenrouten konfigurieren müssen, sollten Sie das Routing zu den Seiten mithilfe der [AddPageRoute-Konvention](#configure-a-page-route) konfigurieren, die weiter unten in diesem Artikel beschrieben wird.

[Anzeigen oder Herunterladen von Beispielcode](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/razor-pages/razor-pages-conventions/sample/) ([Vorgehensweise zum Herunterladen](xref:tutorials/index#how-to-download-a-sample))

::: moniker range="= aspnetcore-2.0"
| Szenario | Dieses Beispiel veranschaulicht Folgendes: |
| -------- | --------------------------- |
| [Model conventions (Modellkonventionen)](#model-conventions)<br><br>Conventions.Add<ul><li>IPageRouteModelConvention</li><li>IPageApplicationModelConvention</li></ul> | Das Hinzufügen einer Routenvorlage und eines Headers zu den Seiten einer App |
| [Konventionen für Seitenroutenaktionen](#page-route-action-conventions)<ul><li>AddFolderRouteModelConvention</li><li>AddPageRouteModelConvention</li><li>AddPageRoute</li></ul> | Das Hinzufügen einer Routenvorlage zu Seiten in einem Ordner und zu einer Einzelseite |
| [Konventionen für Seitenmodellaktionen](#page-model-action-conventions)<ul><li>AddFolderApplicationModelConvention</li><li>AddPageApplicationModelConvention</li><li>ConfigureFilter (Filterklasse, Lambdaausdruck oder Filterzuordnungsinstanz)</li></ul> | Das Hinzufügen eines Headers zu Seiten in einem Ordner, das Hinzufügen eines Headers zu einer einzelnen Seite und das Konfigurieren einer [Filter-Factoy](xref:mvc/controllers/filters#ifilterfactory) zum Hinzufügen eines Headers zu den Seiten einer App. |
| [Standardanbieter Seitenanwendungsmodelle](#replace-the-default-page-app-model-provider) | Das Ersetzen des Standardanbieters für Seitenmodelle, um die Konventionen für Handlernamen zu verändern |
::: moniker-end
::: moniker range=">= aspnetcore-2.1"
| Szenario | Dieses Beispiel veranschaulicht Folgendes: |
| -------- | --------------------------- |
| [Model conventions (Modellkonventionen)](#model-conventions)<br><br>Conventions.Add<ul><li>IPageRouteModelConvention</li><li>IPageApplicationModelConvention</li><li>IPageHandlerModelConvention</li></ul> | Das Hinzufügen einer Routenvorlage und eines Headers zu den Seiten einer App |
| [Konventionen für Seitenroutenaktionen](#page-route-action-conventions)<ul><li>AddFolderRouteModelConvention</li><li>AddPageRouteModelConvention</li><li>AddPageRoute</li></ul> | Das Hinzufügen einer Routenvorlage zu Seiten in einem Ordner und zu einer Einzelseite |
| [Konventionen für Seitenmodellaktionen](#page-model-action-conventions)<ul><li>AddFolderApplicationModelConvention</li><li>AddPageApplicationModelConvention</li><li>ConfigureFilter (Filterklasse, Lambdaausdruck oder Filterzuordnungsinstanz)</li></ul> | Das Hinzufügen eines Headers zu Seiten in einem Ordner, das Hinzufügen eines Headers zu einer einzelnen Seite und das Konfigurieren einer [Filter-Factoy](xref:mvc/controllers/filters#ifilterfactory) zum Hinzufügen eines Headers zu den Seiten einer App. |
| [Standardanbieter Seitenanwendungsmodelle](#replace-the-default-page-app-model-provider) | Das Ersetzen des Standardanbieters für Seitenmodelle, um die Konventionen für Handlernamen zu verändern |
::: moniker-end

Konventionen für Razor-Seiten werden [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc) mithilfe der Erweiterungsmethode [AddRazorPagesOptions](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvcbuilderextensions.addrazorpagesoptions) unter der Dienstklasse in der Klasse `Startup` hinzugefügt. Die folgenden Beispiele der Konvention werden später in diesem Thema erläutert:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddRazorPagesOptions(options =>
            {
                options.Conventions.Add( ... );
                options.Conventions.AddFolderRouteModelConvention("/OtherPages", model => { ... });
                options.Conventions.AddPageRouteModelConvention("/About", model => { ... });
                options.Conventions.AddPageRoute("/Contact", "TheContactPage/{text?}");
                options.Conventions.AddFolderApplicationModelConvention("/OtherPages", model => { ... });
                options.Conventions.AddPageApplicationModelConvention("/About", model => { ... });
                options.Conventions.ConfigureFilter(model => { ... });
                options.Conventions.ConfigureFilter( ... );
            });
}
```

## <a name="model-conventions"></a>Modellkonventionen

Fügen Sie einen Delegaten für [IPageConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageconvention) hinzu, um [Modellkonventionen](xref:mvc/controllers/application-model#conventions) hinzuzufügen, die auf Razor-Seiten anwendbar sind.

**Hinzufügen einer Routenmodellkonvention zu allen Seiten**

Verwenden Sie [Conventions](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.razorpagesoptions.conventions) zum Erstellen und Hinzufügen einer [IPageRouteModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageroutemodelconvention) zur Auflistung der Instanzen von [IPageConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageconvention), die während der Erstellung von Seitenroutenmodellen eingesetzt werden.

Die Beispielanwendung fügt dann zu allen Seiten der App eine `{globalTemplate?}`-Routenvorlage hinzu:

[!code-csharp[](razor-pages-conventions/sample/Conventions/GlobalTemplatePageRouteModelConvention.cs?name=snippet1)]

> [!NOTE]
> Die Eigenschaft `Order` ist für das `AttributeRouteModel` auf `-1` festgelegt. Dadurch wird sichergestellt, dass diese Vorlage in Bezug auf die erste Position für einen Routendatenwert Priorität erhält, wenn nur ein einziger Routenwert angegeben wird. Außerdem wird die Priorität über automatisch generierte Routen für Razor-Seiten sichergestellt. Beispielsweise wird durch die Beispielanwendung weiter unten in diesem Artikel eine `{aboutTemplate?}`-Routenvorlage hinzugefügt. Die Vorlage `{aboutTemplate?}` erhält den `Order` von `1`. Wenn die Seite „Info“ unter `/About/RouteDataValue` angefordert wird, wird „RouteDataValue“ in die Vorlage `RouteData.Values["globalTemplate"]` geladen (`Order = -1`) und nicht in `RouteData.Values["aboutTemplate"]` (`Order = 1`), da die Eigenschaft `Order` auf null festgelegt wurde.

Optionen für Razor-Seiten, z.B. das Hinzufügen von [Conventions (Konventionen)](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.razorpagesoptions.conventions), werden hinzugefügt, wenn MVC der Dienstauflistung in `Startup.ConfigureServices` hinzugefügt wird. In der [Beispiel-App](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/razor-pages/razor-pages-conventions/sample/) finden Sie ein Beispiel hierfür.

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet1)]

Fordern Sie die Seite „Info“ der Beispielanwendung unter `localhost:5000/About/GlobalRouteValue` an, und prüfen Sie das Ergebnis:

![Die Seite „Info“ wird mit einem Routensegment von GlobalRouteValue angefordert. Die gerenderte Seite zeigt, dass der Datenwert für die Route in der OnGet-Methode der Seite erfasst wird.](razor-pages-conventions/_static/about-page-global-template.png)

**Hinzufügen einer App-Modellkonvention zu allen Seiten**

Verwenden Sie [Conventions](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.razorpagesoptions.conventions) zum Erstellen und Hinzufügen einer [IPageApplicationModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageapplicationmodelconvention) zur Auflistung der Instanzen von [IPageConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageconvention), die während der Erstellung von Seitenanwendungsmodellen eingesetzt werden.

Die Beispielanwendung enthält eine `AddHeaderAttribute`-Klasse, damit das Verwenden dieser und anderer Konventionen weiter unten in diesem Thema gezeigt werden kann. Der Klassenkonstruktor akzeptiert eine `name`-Zeichenfolge und ein `values`-Zeichenfolgenarray. Diese Werte werden in seiner `OnResultExecuting`-Methode verwendet, um einen Antwortheader einzurichten. Die Klasse wird im Abschnitt [Seitenmodellaktionskonventionen](#page-model-action-conventions) weiter unten in diesem Artikel erläutert.

Die Beispielanwendung verwendet die Klasse `AddHeaderAttribute`, um den Header `GlobalHeader` zu allen Seiten der Anwendung hinzuzufügen:

[!code-csharp[](razor-pages-conventions/sample/Conventions/GlobalHeaderPageApplicationModelConvention.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet2)]

Fordern Sie die Seite „Info“ der Beispielanwendung unter `localhost:5000/About` an, und überprüfen Sie die Header, um das Ergebnis zu sehen:

![Antwortheader der Seite „Info“ zeigen an, dass „GlobalHeader“ hinzugefügt wurde.](razor-pages-conventions/_static/about-page-global-header.png)

::: moniker range=">= aspnetcore-2.1"
**Hinzufügen einer Handlermodellkonvention zu allen Seiten**

Verwenden Sie [Conventions](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.razorpagesoptions.conventions) zum Erstellen und Hinzufügen einer [IPageHandlerModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipagehandlermodelconvention) zur Auflistung der Instanzen von [IPageConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageconvention), die während der Erstellung von Seitenhandlermodellen eingesetzt werden.

```csharp
public class GlobalPageHandlerModelConvention 
    : IPageHandlerModelConvention
{
    public void Apply(PageHandlerModel model)
    {
        ...
    }
}
```

`Startup.ConfigureServices`:

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
        {
            options.Conventions.Add(new GlobalPageHandlerModelConvention());
        });
```
::: moniker-end

## <a name="page-route-action-conventions"></a>Konventionen für Seitenroutenaktionen

Der Standardanbieter für Routenmodelle, der von [IPageRouteModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageroutemodelprovider) abgeleitet wird, ruft Konventionen auf, die entwickelt wurden, um Erweiterungspunkte zum Konfigurieren von Seitenrouten bereitzustellen.

**Ordnerroutenmodellkonvention**

Verwenden Sie [AddFolderRouteModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageconventioncollection.addfolderroutemodelconvention) zum Erstellen und Hinzufügen von [IPageRouteModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageroutemodelconvention), wodurch für alle Seiten im angegebenen Ordner eine Aktion auf dem [PageRouteModel](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageroutemodel) aufgerufen wird.

Die Beispielanwendung verwendet `AddFolderRouteModelConvention`, um eine `{otherPagesTemplate?}`-Routenvorlage zu den Seiten im Ordner *OtherPages* hinzuzufügen:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet3)]

> [!NOTE]
> Die Eigenschaft `Order` ist für das `AttributeRouteModel` auf `1` festgelegt. Dadurch wird sichergestellt, dass die Vorlage für `{globalTemplate?}`, die weiter oben in diesem Artikel festgelegt wurde, in Bezug auf die erste Position für einen Routendatenwert vorgezogen wird, wenn nur ein einziger Routenwert angegeben wurde. Falls die Seite „Seite1“ unter `/OtherPages/Page1/RouteDataValue` angefordert wird, wird „RouteDataValue“ in die Vorlage `RouteData.Values["globalTemplate"]` geladen (`Order = -1`) und nicht in `RouteData.Values["otherPagesTemplate"]` (`Order = 1`), da die Eigenschaft `Order` auf null festgelegt wurde.

Fordern Sie die Seite „Seite1“ der Beispielanwendung unter `localhost:5000/OtherPages/Page1/GlobalRouteValue/OtherPagesRouteValue` an, und prüfen Sie das Ergebnis:

![Seite1 aus dem Ordner „OtherPages“ wird mit einem Routensegment von GlobalRouteValue und OtherPagesRouteValue angefordert. In der gerenderten Seite wird gezeigt, dass die Datenwerte für die Route in der OnGet-Methode der Seite erfasst werden.](razor-pages-conventions/_static/otherpages-page1-global-and-otherpages-templates.png)

**Seitenroutenmodellkonvention**

Verwenden Sie [AddPageRouteModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageconventioncollection.addpageroutemodelconvention) zum Erstellen und Hinzufügen einer [IPageRouteModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageroutemodelconvention), die für die Seite mit dem angegebenen Namen eine Aktion auf dem [PageRouteModel](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageroutemodel) aufruft.

Die Beispielanwendung verwendet `AddPageRouteModelConvention`, um eine `{aboutTemplate?}`-Routenvorlage zu der Seite „Info“ hinzuzufügen:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet4)]

> [!NOTE]
> Die Eigenschaft `Order` ist für das `AttributeRouteModel` auf `1` festgelegt. Dadurch wird sichergestellt, dass die Vorlage für `{globalTemplate?}`, die weiter oben in diesem Artikel festgelegt wurde, in Bezug auf die erste Position für einen Routendatenwert vorgezogen wird, wenn nur ein einziger Routenwert angegeben wurde. Falls die Seite „Info“ unter `/About/RouteDataValue` angefordert wird, wird „RouteDataValue“ in die Vorlage `RouteData.Values["globalTemplate"]` geladen (`Order = -1`) und nicht in `RouteData.Values["aboutTemplate"]` (`Order = 1`), da die Eigenschaft `Order` auf null festgelegt wurde.

Fordern Sie die Seite „Info“ der Beispielanwendung unter `localhost:5000/About/GlobalRouteValue/AboutRouteValue` an, und prüfen Sie das Ergebnis:

![Die Seite „Info“ wird mit Routensegmenten für GlobalRouteValue und AboutRouteValue angefordert. In der gerenderten Seite wird gezeigt, dass die Datenwerte für die Route in der OnGet-Methode der Seite erfasst werden.](razor-pages-conventions/_static/about-page-global-and-about-templates.png)

## <a name="configure-a-page-route"></a>Konfigurieren einer Seitenroute

Konfigurieren Sie mithilfe von [AddPageRoute](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.addpageroute) eine Route zu einer Seite am angegebenen Seitenpfad. Generierte Links, die auf die Seite verweisen, verwenden die von Ihnen angegebene Route. `AddPageRoute` verwendet die `AddPageRouteModelConvention` zum Aufbauen der Route.

Die Beispielanwendung erstellt für *Contact.cshtml* eine Route zur Seite `/TheContactPage`:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet5)]

Die Seite „Kontakt“ kann auch unter `/Contact` über ihre Standardroute erreicht werden.

Die benutzerdefinierte Route der Beispielanwendung, die zur Seite „Kontakt“ führt, ermöglicht das Verwenden eines optionalen `text`-Routensegments (`{text?}`). Die Seite enthält dieses optionale Segment auch in ihrer `@page`-Anweisung, für den Fall, dass der Besucher über die `/Contact`-Route auf die Seite zugreift:

[!code-cshtml[](razor-pages-conventions/sample/Pages/Contact.cshtml?highlight=1)]

Beachten Sie, dass die URL, die für den Link **Kontakt** in der gerenderten Seite generiert wurde, die aktualisierte Route widerspiegelt:

![Kontaktlink der Beispielanwendung in der Navigationsleiste](razor-pages-conventions/_static/contact-link.png)

![Beim Überprüfen des Kontaktlinks in der gerenderten HTML zeigt sich, dass das HREF-Attribut auf „/TheContactPage“ festgelegt wurde.](razor-pages-conventions/_static/contact-link-source.png)

Sie können die Kontaktseite entweder über deren übliche Route, `/Contact`, oder über die benutzerdefinierte Route, `/TheContactPage`, besuchen. Wenn Sie ein zusätzliches `text`-Routensegment bereitstellen, wird dieses HTML-codierte Segment auf der Seite angezeigt:

![Beispiel für das Bereitstellen eines optionalen „Text“-Routensegments mit „TextValue“ in der URL in der Ansicht des Microsoft Edge-Browsers Die gerenderte Seite zeigt den Segmentwert „Text“ an.](razor-pages-conventions/_static/route-segment-with-custom-route.png)

## <a name="page-model-action-conventions"></a>Konventionen für Seitenmodellaktionen

Der Standardanbieter für Seitenmodelle, der [IPageApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageapplicationmodelprovider) implementiert, ruft Konventionen auf, die entwickelt wurden, um Erweiterungspunkte zum Konfigurieren von Seitenmodellen bereitzustellen. Diese Konventionen sind beim Erstellen und Ändern von Seitenermittlungs- und Verarbeitungsszenarios hilfreich.

In den Beispielen in diesem Abschnitt verwendet die Beispielanwendung eine `AddHeaderAttribute`-Klasse, also ein [ResultFilterAttribute](/dotnet/api/microsoft.aspnetcore.mvc.filters.resultfilterattribute), das einen Antwortheader verwendet:

[!code-csharp[](razor-pages-conventions/sample/Filters/AddHeader.cs?name=snippet1)]

Das Beispiel veranschaulicht mithilfe von Konventionen, wie das Attribut auf alle Seiten in einem Ordner und auf eine Einzelseite angewendet werden kann.

**Ordner-App-Modellkonvention**

Verwenden Sie [AddFolderApplicationModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageconventioncollection.addfolderapplicationmodelconvention) zum Erstellen und Hinzufügen einer [IPageApplicationModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageapplicationmodelconvention), die für alle Seiten in dem angegebenen Ordner eine Aktion in [PageApplicationModel](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageapplicationmodel)-Instanzen aufruft.

Im Beispiel wird die Verwendung der `AddFolderApplicationModelConvention` durch Hinzufügen eines Headers, `OtherPagesHeader`, zu den Seiten im Ordner *OtherPages* der Anwendung veranschaulicht:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet6)]

Fordern Sie die Seite „Seite1“ der Beispielanwendung unter `localhost:5000/OtherPages/Page1` an, und prüfen Sie die Header, um das Ergebnis zu sehen:

![Antwortheader der Seite „OtherPages/Seite1“ zeigt, dass der OtherPagesHeader hinzugefügt wurde.](razor-pages-conventions/_static/page1-otherpages-header.png)

**Seiten-App-Modellkonvention**

Verwenden Sie [AddPageApplicationModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageconventioncollection.addpageapplicationmodelconvention) zum Erstellen und Hinzufügen einer [IPageApplicationModelConvention](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.ipageapplicationmodelconvention), die für die Seite mit dem angegebenen Namen eine Aktion im [PageApplicationModel](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageapplicationmodel) aufruft.

Im Beispiel wird das Verwenden von `AddPageApplicationModelConvention` durch Hinzufügen eines Headers, `AboutHeader`, auf der Seite „Info“ veranschaulicht:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet7)]

Fordern Sie die Seite „Info“ der Beispielanwendung unter `localhost:5000/About` an, und überprüfen Sie die Header, um das Ergebnis zu sehen:

![Antwortheader der Seite „Info“ zeigen, dass AboutHeader hinzugefügt wurde.](razor-pages-conventions/_static/about-page-about-header.png)

**Konfigurieren eines Filters**

Der angegebene Filter, der angewendet werden soll, wird mit [ConfigureFilter](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.configurefilter) konfiguriert. Sie können Filterklassen selbst implementieren. In der Beispielanwendung wird jedoch gezeigt, wie Sie einen Filter in einen Lambdaausdruck implementieren. Dieser wird dann im Hintergrund als Zuordnungsinstanz implementiert, die einen Filter zurückgibt:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet8)]

Das Seiten-App-Modell wird verwendet, um den relativen Pfad für Segmente zu überprüfen, die zur Seite „Seite2“ im Ordner *OtherPages* führen. Wenn die Bedingung erfüllt ist, wird ein Header hinzugefügt. Wenn dies nicht der Fall ist, wird der `EmptyFilter` angewendet.

`EmptyFilter` ist ein [Aktionsfilter](xref:mvc/controllers/filters#action-filters). Wenn der Pfad `OtherPages/Page2` nicht enthält, hat der `EmptyFilter`, wie vorgesehen, keine Funktion, da Aktionsfilter von Razor Pages ignoriert werden.

Fordern Sie die Seite „Seite2“ der Beispielanwendung unter `localhost:5000/OtherPages/Page2` an, und prüfen Sie die Header, um das Ergebnis zu sehen:

![Der Header „OtherPagesPage2Header“ wird zur Antwort für „Seite2“ hinzugefügt.](razor-pages-conventions/_static/page2-filter-header.png)

**Konfigurieren einer Filterzuordnungsinstanz**

Mit [ConfigureFilter](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.configurefilter?view=aspnetcore-2.0#Microsoft_Extensions_DependencyInjection_PageConventionCollectionExtensions_ConfigureFilter_Microsoft_AspNetCore_Mvc_ApplicationModels_PageConventionCollection_System_Func_Microsoft_AspNetCore_Mvc_ApplicationModels_PageApplicationModel_Microsoft_AspNetCore_Mvc_Filters_IFilterMetadata__) wird die angegebene Zuordnungsinstanz so konfiguriert, dass sie [Filter](xref:mvc/controllers/filters) auf alle Razor Pages anwendet.

Die Beispielanwendung bietet Ihnen die Möglichkeit des beispielhaften Verwendens einer [Filterzuordnungsinstanz](xref:mvc/controllers/filters#ifilterfactory) durch Hinzufügen des Headers `FilterFactoryHeader` zu den Seiten der Anwendung, der zwei Werte besitzt:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet9)]

*AddHeaderWithFactory.cs*:

[!code-csharp[](razor-pages-conventions/sample/Factories/AddHeaderWithFactory.cs?name=snippet1)]

Fordern Sie die Seite „Info“ der Beispielanwendung unter `localhost:5000/About` an, und überprüfen Sie die Header, um das Ergebnis zu sehen:

![Die Antwortheader der Seite „Info“ zeigen, dass zwei Header des Typs „FilterFactoryHeader“ hinzugefügt wurden.](razor-pages-conventions/_static/about-page-filter-factory-header.png)

## <a name="replace-the-default-page-app-model-provider"></a>Ersetzen der Standardanbieter für Modelle für Seitenanwendungen

Razor Pages verwenden die Schnittstelle `IPageApplicationModelProvider` zum Erstellen von [DefaultPageApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.internal.defaultpageapplicationmodelprovider). Sie können vom Standardanbieter für Modelle das übernehmen, was Sie benötigen, um Ihre eigene Implementierungslogik für die Ermittlung und Verarbeitung von Handlern bereitstellen zu können. Die Standardimplementierung ([Referenzquelle](https://github.com/aspnet/Mvc/blob/rel/2.0.1/src/Microsoft.AspNetCore.Mvc.RazorPages/Internal/DefaultPageApplicationModelProvider.cs)) erstellt Konventionen zur Benennung von *unbenannten* und *benannten* Handlern, was nachfolgend genauer beschrieben wird.

**Standardmethoden unbenannter Handler**

Handlermethoden für HTTP-Verben (Methoden „unbenannter“ Handler) folgen der Konvention `On<HTTP verb>[Async]`. Das Anfügen von `Async` ist optional, wird jedoch für asynchrone Methoden empfohlen.

| Methode eines unbenannten Handlers     | Vorgang                      |
| -------------------------- | ------------------------------ |
| `OnGet`/`OnGetAsync`       | Initialisieren des Seitenzustands     |
| `OnPost`/`OnPostAsync`     | Verarbeiten von POST-Anforderungen          |
| `OnDelete`/`OnDeleteAsync` | Verarbeiten von DELETE-Anforderungen&#8224;. |
| `OnPut`/`OnPutAsync`       | Verarbeiten von PUT-Anforderungen&#8224;.    |
| `OnPatch`/`OnPatchAsync`   | Verarbeiten von PATCH-Anforderungen&#8224;.  |

&#8224;Zum Durchführen von API-Aufrufen der Seite

**Standardmethoden für benannte Handler**

Vom Entwickler zur Verfügung gestellte Handlermethoden (Methoden „benannter“ Handler) folgen einer ähnlichen Konvention. Der Name des Handlers wird nach dem HTTP-Verb oder zwischen HTTP-Verb und `Async` angezeigt: `On<HTTP verb><handler name>[Async]`. Das Anfügen von `Async` ist optional, wird jedoch für asynchrone Methoden empfohlen. Methoden, die Nachrichten verarbeiten, könnten beispielsweise so benannt sein, wie in der folgenden Tabelle dargestellt.

| Beispielmethode eines benannten Handlers             | Beispielvorgang        |
| ---------------------------------------- | ------------------------ |
| `OnGetMessage`/`OnGetMessageAsync`       | Abrufen einer Nachricht        |
| `OnPostMessage`/`OnPostMessageAsync`     | POST a message (Versenden einer Nachricht)          |
| `OnDeleteMessage`/`OnDeleteMessageAsync` | DELETE a message (Löschen einer Nachricht)&#8224;. |
| `OnPutMessage`/`OnPutMessageAsync`       | PUT a message (Platzieren einer Nachricht)&#8224;    |
| `OnPatchMessage`/`OnPatchMessageAsync`   | PATCH a message (Patch für eine Nachricht anwenden)&#8224;.  |

&#8224;Zum Durchführen von API-Aufrufen der Seite

**Anpassen der Namen von Handlermethoden**

Angenommen, Sie möchten unbenannte und benannte Handlermethoden anhand eines anderen Schemas benennen. Ein alternatives Benennungsschema besteht darin, die Methodennamen nicht mit „On“ beginnen zu lassen, sondern das erste Wortsegment zum Bestimmen des HTTP-Verbs zu verwenden. Sie können auch andere Änderungen vornehmen, z.B. die Verben für DELETE, PUT und PATCH in POST konvertieren. Solch ein Schema liefert die Methodennamen, die in der folgenden Tabelle aufgelistet sind.

| Handlermethode                       | Vorgang                      |
| ------------------------------------ | ------------------------------ |
| `Get`                                | Initialisieren des Seitenzustands     |
| `Post`/`PostAsync`                   | Verarbeiten von POST-Anforderungen          |
| `Delete`/`DeleteAsync`               | Verarbeiten von DELETE-Anforderungen&#8224;. |
| `Put`/`PutAsync`                     | Verarbeiten von PUT-Anforderungen&#8224;.    |
| `Patch`/`PatchAsync`                 | Verarbeiten von PATCH-Anforderungen&#8224;.  |
| `GetMessage`                         | Abrufen einer Nachricht              |
| `PostMessage`/`PostMessageAsync`     | POST a message (Versenden einer Nachricht)                |
| `DeleteMessage`/`DeleteMessageAsync` | POST a message to delete (Versenden einer zu löschenden Nachricht)      |
| `PutMessage`/`PutMessageAsync`       | POST a message to put (Versenden einer zu platzierenden Nachricht)         |
| `PatchMessage`/`PatchMessageAsync`   | POST a message to patch (Versenden einer Nachricht, für die ein Patch angewendet werden soll)       |

&#8224;Zum Durchführen von API-Aufrufen der Seite

Wenn Sie dieses Schema erstellen möchten, muss von der Klasse `DefaultPageApplicationModelProvider` geerbt werden und die Methode [CreateHandlerModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.internal.defaultpageapplicationmodelprovider.createhandlermodel) muss überschrieben werden, damit eine benutzerdefinierte Logik zum Auflösen von [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel)-Handlernamen bereitgestellt werden kann. In der Beispielanwendung wird anhand der Klasse `CustomPageApplicationModelProvider` gezeigt, wie dies funktioniert:

[!code-csharp[](razor-pages-conventions/sample/CustomPageApplicationModelProvider.cs?name=snippet1&highlight=1-2,45-46,64-68,78-85,87,92,106)]

Die Klasse bietet u.a. folgende Vorteile:

* Die Klasse erbt von `DefaultPageApplicationModelProvider`.
* Die `TryParseHandlerMethod` verarbeitet einen Handler, um beim Erstellen von `PageHandlerModel` das HTTP-Verb (`httpMethod`) sowie den Namen des benannten Handlers (`handlerName`) zu bestimmen.
  * `Async`-Postfixe werden ignoriert, falls vorhanden.
  * Groß- und Kleinschreibung wird verwendet, um das HTTP-Verb aus dem Methodennamen herauszulesen.
  * Wenn der Methodenname (ohne `Async`) dem Namen des HTTP-Verbs gleicht, ist kein benannter Handler vorhanden. Der `handlerName` ist auf `null` festgelegt, und der Methodenname lautet `Get`, `Post`, `Delete`, `Put` oder `Patch`.
  * Wenn der Methodenname (ohne `Async`) länger als der Name des HTTP-Verbs ist, ist ein benannter Handler vorhanden. `handlerName` wird auf `<method name (less 'Async', if present)>` festgelegt. Beispielsweise geben sowohl „GetMessage“ als auch „GetMessageAsync“ den Handlernamen „GetMessage“ zurück.
  * Die HTTP-Verben DELETE, PUT und PATCH werden in POST konvertiert.

So registrieren Sie den `CustomPageApplicationModelProvider` in der Klasse `Startup`:

[!code-csharp[](razor-pages-conventions/sample/Startup.cs?name=snippet10)]

Das Seitenmodell in der Datei *Index.cshtml.cs* zeigt, wie die üblichen Namenskonventionen für Handlermethoden für Seiten in der Anwendung geändert werden. Das Präfix „On“, das in Razor Pages üblicherweise bei der Benennung verwendet wird, wird entfernt. Die Methode, die den Seitenstatus initialisiert, heißt jetzt `Get`. Sie können sehen, dass diese Konvention in der ganzen Anwendung verwendet wird, wenn Sie eines der Seitenmodelle für eine der Seiten öffnen.

Alle anderen Methoden beginnen mit dem HTTP-Verb, das ihre Verarbeitung beschreibt. Die beiden Methoden, die mit `Delete` beginnen, würden normalerweise wie DELETE-HTTP-Verben behandelt werden, aber die Logik der `TryParseHandlerMethod` legt für beide Handler das Verb explizit auf POST fest.

Beachten Sie, dass `Async` optional ist, wenn es um `DeleteAllMessages` und `DeleteMessageAsync` geht. Beides sind asynchrone Methoden. Es steht Ihnen frei, das Postfix `Async` zu verwenden, allerdings wird empfohlen, es zu verwenden. `DeleteAllMessages` wird hier lediglich zur Veranschaulichung verwendet. Es wird jedoch empfohlen, dass Sie einer solchen Methode den Namen `DeleteAllMessagesAsync` geben. Dies wirkt sich nicht auf die Verarbeitung der Implementierung der Beispielanwendung aus, aber durch das Verwenden des Postfixes `Async` wird die Tatsache hervorgehoben, dass es sich um eine asynchrone Methode handelt.

[!code-csharp[](razor-pages-conventions/sample/Pages/Index.cshtml.cs?name=snippet1&highlight=1,6,16,29)]

Beachten Sie, dass die in *Index.cshtml* bereitgestellten Handlernamen den Handlermethoden `DeleteAllMessages` und `DeleteMessageAsync` entsprechen:

[!code-cshtml[](razor-pages-conventions/sample/Pages/Index.cshtml?range=29-60&highlight=7-8,24-25)]

Das Postfix `Async` im Namen der Handlermethode `DeleteMessageAsync` wird durch die `TryParseHandlerMethod` ausgeklammert, die der Handlerzuordnung von POST-Anforderung und entsprechender Methode dient. Der `asp-page-handler`-Name von `DeleteMessage` wird der Handlermethode `DeleteMessageAsync` zugeordnet.

## <a name="mvc-filters-and-the-page-filter-ipagefilter"></a>Die MVC-Filter und der Seitenfilter (IPageFilter)

MVC-[Aktionsfilter](xref:mvc/controllers/filters#action-filters) werden von Razor Pages ignoriert, da diese Handlermethoden verwenden. Sie können jedoch andere Arten von MVC-Filtern verwenden: [Autorisierung](xref:mvc/controllers/filters#authorization-filters), [Ausnahme](xref:mvc/controllers/filters#exception-filters), [Ressource](xref:mvc/controllers/filters#resource-filters) und [Ergebnis](xref:mvc/controllers/filters#result-filters). Weitere Informationen finden Sie im Thema [Filter](xref:mvc/controllers/filters).

Der Seitenfilter ([IPageFilter](/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter)) ist ein Filter, der auf Razor Pages anwendbar ist. Weitere Informationen finden Sie unter [Filtermethoden für Razor-Seiten](xref:mvc/razor-pages/filter).

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Autorisierungskonventionen für Razor Pages](xref:security/authorization/razor-pages-authorization)

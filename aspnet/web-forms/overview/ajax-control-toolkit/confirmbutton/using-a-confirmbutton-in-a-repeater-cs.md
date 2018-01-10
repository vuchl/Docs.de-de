---
uid: web-forms/overview/ajax-control-toolkit/confirmbutton/using-a-confirmbutton-in-a-repeater-cs
title: Verwenden eine ConfirmButton In ein Repeater (c#) | Microsoft Docs
author: wenz
description: "Der Extender ConfirmButton im AJAX Control Toolkit erstellt eine Ja/keine Popupfenster, wenn der Benutzer auf eine Schaltfläche klickt (einschließlich LinkButton-Steuerelement). Es ist Ja nur, wenn..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: a973ed3e-400c-4925-ace2-0b086b479301
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/confirmbutton/using-a-confirmbutton-in-a-repeater-cs
msc.type: authoredcontent
ms.openlocfilehash: 12fd3e4e58e6a73720ade38372caff496f7f2c97
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
<a name="using-a-confirmbutton-in-a-repeater-c"></a><span data-ttu-id="94386-104">Verwenden eine ConfirmButton In ein Repeater (c#)</span><span class="sxs-lookup"><span data-stu-id="94386-104">Using a ConfirmButton In a Repeater (C#)</span></span>
====================
<span data-ttu-id="94386-105">durch [Christian Wenz](https://github.com/wenz)</span><span class="sxs-lookup"><span data-stu-id="94386-105">by [Christian Wenz](https://github.com/wenz)</span></span>

<span data-ttu-id="94386-106">[Herunterladen von Code](http://download.microsoft.com/download/8/6/d/86dea6c6-bb92-4fa6-aa14-f8c0f82100f5/ConfirmButton1.cs.zip) oder [PDF herunterladen](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/confirmbutton1CS.pdf)</span><span class="sxs-lookup"><span data-stu-id="94386-106">[Download Code](http://download.microsoft.com/download/8/6/d/86dea6c6-bb92-4fa6-aa14-f8c0f82100f5/ConfirmButton1.cs.zip) or [Download PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/confirmbutton1CS.pdf)</span></span>

> <span data-ttu-id="94386-107">Der Extender ConfirmButton im AJAX Control Toolkit erstellt eine Ja/keine Popupfenster, wenn der Benutzer auf eine Schaltfläche klickt (einschließlich LinkButton-Steuerelement).</span><span class="sxs-lookup"><span data-stu-id="94386-107">The ConfirmButton extender in the AJAX Control Toolkit creates a Yes/No popup when the user clicks on a button (including LinkButton control).</span></span> <span data-ttu-id="94386-108">Nur wird Wenn Ja geklickt wird, die Schaltfläche Aktion ausgeführt, andernfalls abgebrochen.</span><span class="sxs-lookup"><span data-stu-id="94386-108">Only if Yes is clicked, the button's action is executed, otherwise cancelled.</span></span> <span data-ttu-id="94386-109">Dies ist auch möglich, in ein Repeater.</span><span class="sxs-lookup"><span data-stu-id="94386-109">This is also possible in a repeater.</span></span>


## <a name="overview"></a><span data-ttu-id="94386-110">Übersicht</span><span class="sxs-lookup"><span data-stu-id="94386-110">Overview</span></span>

<span data-ttu-id="94386-111">Der Extender ConfirmButton im AJAX Control Toolkit erstellt eine Ja/keine Popupfenster, wenn der Benutzer auf eine Schaltfläche klickt (einschließlich LinkButton-Steuerelement).</span><span class="sxs-lookup"><span data-stu-id="94386-111">The ConfirmButton extender in the AJAX Control Toolkit creates a Yes/No popup when the user clicks on a button (including LinkButton control).</span></span> <span data-ttu-id="94386-112">Nur wird Wenn Ja geklickt wird, die Schaltfläche Aktion ausgeführt, andernfalls abgebrochen.</span><span class="sxs-lookup"><span data-stu-id="94386-112">Only if Yes is clicked, the button's action is executed, otherwise cancelled.</span></span> <span data-ttu-id="94386-113">Dies ist auch möglich, in ein Repeater.</span><span class="sxs-lookup"><span data-stu-id="94386-113">This is also possible in a repeater.</span></span>

## <a name="steps"></a><span data-ttu-id="94386-114">Schritte</span><span class="sxs-lookup"><span data-stu-id="94386-114">Steps</span></span>

<span data-ttu-id="94386-115">Erstens ist eine Datenquelle erforderlich.</span><span class="sxs-lookup"><span data-stu-id="94386-115">First of all, a data source is required.</span></span> <span data-ttu-id="94386-116">Dieses Beispiel verwendet die AdventureWorks-Datenbank und die Microsoft SQL Server 2005 Express Edition.</span><span class="sxs-lookup"><span data-stu-id="94386-116">This sample uses the AdventureWorks database and the Microsoft SQL Server 2005 Express Edition.</span></span> <span data-ttu-id="94386-117">Die Datenbank ist ein optionaler Teil einer Visual Studio-Installation (einschließlich express Edition) und steht auch als separater Download unter [https://go.microsoft.com/fwlink/?LinkId=64064](https://go.microsoft.com/fwlink/?LinkId=64064).</span><span class="sxs-lookup"><span data-stu-id="94386-117">The database is an optional part of a Visual Studio installation (including express edition) and is also available as a separate download under [https://go.microsoft.com/fwlink/?LinkId=64064](https://go.microsoft.com/fwlink/?LinkId=64064).</span></span> <span data-ttu-id="94386-118">Die AdventureWorks-Datenbank ist Teil der SQL Server 2005 Samples and Sample Databases (download [https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang = En](https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en)).</span><span class="sxs-lookup"><span data-stu-id="94386-118">The AdventureWorks database is part of the SQL Server 2005 Samples and Sample Databases (download at [https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en](https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en)).</span></span> <span data-ttu-id="94386-119">Die einfachste Möglichkeit zum Einrichten der Datenbank ist die Verwendung der Microsoft SQL Server Management Studio Express ([Https://www.microsoft.com/downloads/details.aspx? FamilyID = c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang = En](https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en)), und fügen die `AdventureWorks.mdf` Datenbankdatei.</span><span class="sxs-lookup"><span data-stu-id="94386-119">The easiest way to set the database up is to use the Microsoft SQL Server Management Studio Express ([https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en](https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en)) and attach the `AdventureWorks.mdf` database file.</span></span>

<span data-ttu-id="94386-120">In diesem Beispiel gehen wir davon aus, dass die Instanz von SQL Server 2005 Express Edition aufgerufen wird `SQLEXPRESS` und befindet sich auf dem gleichen Computer wie der Webserver; Dies ist auch die Standardeinstellungen.</span><span class="sxs-lookup"><span data-stu-id="94386-120">For this sample, we assume that the instance of the SQL Server 2005 Express Edition is called `SQLEXPRESS` and resides on the same machine as the web server; this is also the default setup.</span></span> <span data-ttu-id="94386-121">Wenn Ihr Setup abweicht, müssen Sie passen Sie die Verbindungsinformationen für die Datenbank.</span><span class="sxs-lookup"><span data-stu-id="94386-121">If your setup differs, you have to adapt the connection information for the database.</span></span>

<span data-ttu-id="94386-122">Um die Funktionalität von ASP.NET AJAX und das Steuerelement-Toolkit aktivieren die `ScriptManager` Steuerelement an einer beliebigen Stelle auf der Seite versetzt werden muss (jedoch innerhalb der `<form>` Element):</span><span class="sxs-lookup"><span data-stu-id="94386-122">In order to activate the functionality of ASP.NET AJAX and the Control Toolkit, the `ScriptManager` control must be put anywhere on the page (but within the `<form>` element):</span></span>

[!code-aspx[Main](using-a-confirmbutton-in-a-repeater-cs/samples/sample1.aspx)]

<span data-ttu-id="94386-123">Klicken Sie dann, ist eine Datenquelle erforderlich.</span><span class="sxs-lookup"><span data-stu-id="94386-123">Then, a data source is required.</span></span> <span data-ttu-id="94386-124">Der Einfachheit halber werden nur die ersten fünf Einträge in 'AdventureWorks Lieferanten Tabelle abgerufen.</span><span class="sxs-lookup"><span data-stu-id="94386-124">For the sake of simplicity, only the first five entries in AdventureWorks' Vendors table are retrieved.</span></span> <span data-ttu-id="94386-125">Beachten Sie, dass bei Verwendung des Visual Studio-Assistenten zum Erstellen der Datenquelle, den Tabellennamen (`Vendors`) wird derzeit nicht ordnungsgemäß mit dem Präfix `Purchasing`.</span><span class="sxs-lookup"><span data-stu-id="94386-125">Note that when using the Visual Studio wizard to create the data source, the table name (`Vendors`) is currently not correctly prefixed with `Purchasing`.</span></span> <span data-ttu-id="94386-126">Das folgende Markup ist die richtige PIN:</span><span class="sxs-lookup"><span data-stu-id="94386-126">The following markup is the correct one:</span></span>

[!code-aspx[Main](using-a-confirmbutton-in-a-repeater-cs/samples/sample2.aspx)]

<span data-ttu-id="94386-127">Diese Datenquelle kann dann in ein Repeater verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="94386-127">This data source can then be used within a repeater.</span></span> <span data-ttu-id="94386-128">Wie üblich die `DataBinder.Eval()` Methode ruft Daten aus der Datenquelle ab.</span><span class="sxs-lookup"><span data-stu-id="94386-128">As usual, the `DataBinder.Eval()` method retrieves data from the data source.</span></span> <span data-ttu-id="94386-129">Die `ConfirmButtonExtender` Steuerelement muss dann platziert werden, innerhalb der `<ItemTemplate>` Abschnitt des wiederholungsmoduls, damit er für jeden Eintrag in der Datenquelle angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="94386-129">The `ConfirmButtonExtender` control must then be placed within the `<ItemTemplate>` section of the repeater so that it appears for every entry in the data source.</span></span>

[!code-aspx[Main](using-a-confirmbutton-in-a-repeater-cs/samples/sample3.aspx)]


<span data-ttu-id="94386-130">[![Die Schaltfläche "bestätigen" wird neben jeder Eintrag aus der Datenquelle angezeigt.](using-a-confirmbutton-in-a-repeater-cs/_static/image2.png)](using-a-confirmbutton-in-a-repeater-cs/_static/image1.png)</span><span class="sxs-lookup"><span data-stu-id="94386-130">[![The confirm button appears next to each entry from the data source](using-a-confirmbutton-in-a-repeater-cs/_static/image2.png)](using-a-confirmbutton-in-a-repeater-cs/_static/image1.png)</span></span>

<span data-ttu-id="94386-131">Die Schaltfläche "bestätigen" wird neben jeder Eintrag aus der Datenquelle angezeigt ([klicken Sie hier, um das Bild in voller Größe angezeigt](using-a-confirmbutton-in-a-repeater-cs/_static/image3.png))</span><span class="sxs-lookup"><span data-stu-id="94386-131">The confirm button appears next to each entry from the data source ([Click to view full-size image](using-a-confirmbutton-in-a-repeater-cs/_static/image3.png))</span></span>

>[!div class="step-by-step"]
[<span data-ttu-id="94386-132">Nächste</span><span class="sxs-lookup"><span data-stu-id="94386-132">Next</span></span>](using-a-confirmbutton-in-a-repeater-vb.md)
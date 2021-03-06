---
title: Problembehandlung bei ASP.NET Core in IIS
author: guardrex
description: Erfahren Sie, wie Sie Probleme mit IIS-Bereitstellungen (IIS = Internet Information Services) von ASP.NET Core-Apps diagnostizieren können.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/iis/troubleshoot
ms.openlocfilehash: e44892d2022ca1a176cee9d027e220e196c6572d
ms.sourcegitcommit: 48beecfe749ddac52bc79aa3eb246a2dcdaa1862
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/22/2018
ms.locfileid: "30073892"
---
# <a name="troubleshoot-aspnet-core-on-iis"></a>Problembehandlung bei ASP.NET Core in IIS

Von [Luke Latham](https://github.com/guardrex)

Dieser Artikel enthält Anweisungen zum Diagnostizieren eines Startproblems der ASP.NET Core-App beim Hosten mithilfe von [Internet Information Services (IIS)](/iis). Die in diesem Artikel enthaltenen Informationen gelten für das Hosting in IIS unter Windows Server und Windows Desktop.

In Visual Studio entspricht ein ASP.NET Core-Projekt standardmäßig dem [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)-Hosting während des Debuggens. Ein *502.5: Prozessfehler*, der beim lokalen Debuggen auftritt, kann mithilfe der Hinweise in diesem Thema behoben werden.

Weitere Themen zur Problembehandlung:

[Problembehandlung bei ASP.NET Core in Azure App Service](xref:host-and-deploy/azure-apps/troubleshoot)  
Obwohl App Service das [ASP.NET Core-Modul](xref:fundamentals/servers/aspnet-core-module) und IIS für das Hosten von Apps verwendet, finden Sie im dedizierten Thema weitere, für App Service spezifische Anweisungen.

[Behandeln von Fehlern](xref:fundamentals/error-handling)  
Informationen zur Behandlung von Fehlern in ASP.NET Core-Apps während der Bereitstellung auf einem lokalen System.

[Lernen Sie das Debuggen mit Visual Studio](/visualstudio/debugger/getting-started-with-the-debugger)  
In diesem Thema werden die Features des Visual Studio-Debuggers vorgestellt.

## <a name="app-startup-errors"></a>App-Startfehler

**502.5: Prozessfehler**  
Der Workerprozess schlägt fehl. Die App wird nicht gestartet.

Das ASP.NET Core-Modul kann den Workerprozess nicht starten. Die Ursache für einen Fehler beim Starten eines Prozesses kann in der Regel über Einträge im [Anwendungsereignisprotokoll](#application-event-log) und im [stdout-Protokoll des ASP.NET Core-Moduls](#aspnet-core-module-stdout-log) ermittelt werden.

Die Fehlerseite *502.5: Prozessfehler* wird zurückgegeben, wenn ein falsch konfiguriertes Hosting oder eine falsch konfigurierte App bewirkt, dass der Workerprozess fehlschlägt:

![Browserfenster mit der Seite „502.5: Prozessfehler“](troubleshoot/_static/process-failure-page.png)

**500: Interner Serverfehler**  
Die App wird gestartet, aber ein Fehler verhindert, dass der Server auf die Anforderung eingeht.

Dieser Fehler tritt im Code der App während des Starts oder bei der Erstellung einer Antwort auf. Die Antwort enthält möglicherweise keinen Inhalt oder die Antwort wird als *500: Interner Serverfehler* im Browser angezeigt. Das Anwendungsereignisprotokoll gibt normalerweise an, dass die Anwendung normal gestartet wurde. Aus Sicht des Servers ist dies richtig. Die App wurde gestartet, sie kann jedoch keine gültige Antwort generieren. [Führen Sie die App in einer Eingabeaufforderung](#run-the-app-at-a-command-prompt) auf dem Server aus oder [aktivieren Sie das stdout-Protokoll des ASP.NET Core-Moduls](#aspnet-core-module-stdout-log), um das Problem zu beheben.

**Verbindungszurücksetzung**

Falls ein Fehler auftritt, nachdem die Header gesendet wurden, ist es zu spät für den Server, einen **500: Interner Serverfehler** zu senden, wenn ein Fehler auftritt. Dies ist häufig der Fall, wenn während der Serialisierung komplexer Objekte für eine Antwort ein Fehler auftritt. Diese Art von Fehler wird angezeigt, wenn ein *Verbindungszurücksetzungsfehler* auf dem Client auftritt. Mithilfe der [Anwendungsprotokollierung](xref:fundamentals/logging/index) können diese Fehlertypen behoben werden.

## <a name="default-startup-limits"></a>Standardstarteinschränkungen

Das ASP.NET Core-Modul wurde mit dem Standardwert 120 Sekunden für das *StartupTimeLimit* konfiguriert. Wenn der Standardwert beibehalten wird, dauert der Start einer App möglicherweise bis zu zwei Minuten. Erst danach kann das Modul einen Prozessfehler protokollieren. Weitere Informationen zum Konfigurieren des Moduls finden Sie unter [Attribute des aspNetCore-Elements](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element).

## <a name="troubleshoot-app-startup-errors"></a>Problembehandlung bei App-Startfehlern

### <a name="application-event-log"></a>Anwendungsereignisprotokoll

Greifen Sie auf das Anwendungsereignisprotokoll zu:

1. Öffnen Sie das Menü „Start“, suchen Sie nach der **Ereignisanzeige**, und wählen Sie anschließend die App **Ereignisanzeige** aus.
1. Öffnen Sie unter **Ereignisanzeige** den Knoten **Windows-Protokolle**.
1. Wählen Sie **Anwendung** aus, um das Anwendungsereignisprotokoll zu öffnen.
1. Suchen Sie nach Fehlern, die mit der fehlerhaften App im Zusammenhang stehen. Fehler weisen in der Spalte *Quelle* den Wert *IIS AspNetCore-Modul* oder *IIS Express AspNetCore-Modul* auf.

### <a name="run-the-app-at-a-command-prompt"></a>Ausführen der App in einer Eingabeaufforderung

Viele Startfehler erzeugen keine nützlichen Informationen im Anwendungsereignisprotokoll. Sie können die Ursache für einige Fehler ermitteln, indem Sie die App in einer Eingabeaufforderung auf dem Hostsystem ausführen.

**Framework-abhängige Bereitstellung**

Wenn es sich bei der App um eine [Framework-abhängige Bereitstellung handelt](/dotnet/core/deploying/#framework-dependent-deployments-fdd):

1. Navigieren Sie in einer Eingabeaufforderung zum Bereitstellungsordner und führen Sie die App aus, indem Sie die Assembly der App mit *dotnet.exe* ausführen. Ersetzen Sie in folgendem Befehl den Namen der Assembly der App durch \<assembly_name>: `dotnet .\<assembly_name>.dll`.
1. Die Konsolenausgabe der App, die Fehler anzeigt, wird in das Konsolenfenster geschrieben.
1. Wenn der Fehler während einer Anforderung an die App auftritt, führen Sie eine Anforderung an den Host und Port aus, auf dem Kestrel empfangsbereit ist. Führen Sie unter Verwendung der Standardhosts und -ports eine Anforderung für `http://localhost:5000/` aus. Wenn die App normalerweise auf die Kestrel-Endpunktadresse reagiert, hängt das Problem wahrscheinlich eher mit der Reverseproxykonfiguration und weniger mit der App selbst zusammen.

**Eigenständige Bereitstellung**

Wenn es sich bei der App um eine [eigenständige Bereitstellung](/dotnet/core/deploying/#self-contained-deployments-scd) handelt:

1. Navigieren Sie in einer Eingabeaufforderung zum Bereitstellungsordner und führen Sie die ausführbaren Dateien der App aus. Ersetzen Sie in folgendem Befehl den Namen der Assembly der App durch \<assembly_name>: `<assembly_name>.exe`.
1. Die Konsolenausgabe der App, die Fehler anzeigt, wird in das Konsolenfenster geschrieben.
1. Wenn der Fehler während einer Anforderung an die App auftritt, führen Sie eine Anforderung an den Host und Port aus, auf dem Kestrel empfangsbereit ist. Führen Sie unter Verwendung der Standardhosts und -ports eine Anforderung für `http://localhost:5000/` aus. Wenn die App normalerweise auf die Kestrel-Endpunktadresse reagiert, hängt das Problem wahrscheinlich eher mit der Reverseproxykonfiguration und weniger mit der App selbst zusammen.

### <a name="aspnet-core-module-stdout-log"></a>stdout-Protokoll des ASP.NET Core-Moduls

So aktivieren Sie stdout-Protokolle und zeigen diese an:

1. Navigieren Sie zum Bereitstellungsordner der Website auf dem Hostsystem.
1. Erstellen Sie den Ordner *logs*, wenn dieser nicht vorhanden ist. Anweisungen zum Aktivieren von MSBuild für die automatische Erstellung des Ordners *logs* in der Bereitstellung finden Sie im Thema [Verzeichnisstruktur](xref:host-and-deploy/directory-structure).
1. Bearbeiten Sie die Datei *web.config*. Legen Sie **stdoutLogEnabled** auf `true` fest, und ändern Sie den Pfad von **stdoutLogFile** so, dass auf den Ordner *logs* verwiesen wird (Beispiel: `.\logs\stdout`). `stdout` im Pfad ist das Präfix des Protokolldateinamens. Ein Zeitstempel, eine Prozess-ID und eine Dateierweiterung werden bei der Protokollerstellung automatisch hinzugefügt. Mit `stdout` als Präfix des Dateinamens wird eine typische Protokolldatei als *stdout_20180205184032_5412.log* benannt. 
1. Speichern Sie die aktualisierte Datei *web.config*.
1. Führen Sie eine Anforderung an die App aus.
1. Navigieren Sie zu dem Ordner *logs*. Suchen und öffnen Sie das aktuelle stdout-Protokoll.
1. Untersuchen Sie das Protokoll auf Fehler.

**Wichtig** Deaktivieren Sie die stdout-Protokollierung, wenn die Problembehandlung abgeschlossen ist.

1. Bearbeiten Sie die Datei *web.config*.
1. Legen Sie **stdoutLogEnabled** auf `false` fest.
1. Speichern Sie die Datei.

> [!WARNING]
> Wenn das stdout-Protokoll nicht deaktiviert wird, können App- oder Serverfehler auftreten. Für die Protokollgröße oder die Anzahl von erstellten Protokolldateien ist kein Grenzwert festgelegt.
>
> Verwenden Sie für die routinemäßige Protokollierung in einer ASP.NET Core-App eine Protokollierungsbibliothek, die die Protokolldateigröße beschränkt und die Protokolle rotiert. Weitere Informationen finden Sie im Artikel zur [Protokollierung von Drittanbietern](xref:fundamentals/logging/index#third-party-logging-providers).

## <a name="enabling-the-developer-exception-page"></a>Aktivieren der Seite mit Ausnahmen für Entwickler

Die `ASPNETCORE_ENVIRONMENT` [Umgebungsvariable kann zur Datei web.config hinzugefügt werden](xref:host-and-deploy/aspnet-core-module#setting-environment-variables), um die App in der Entwicklungsumgebung auszuführen. Solange die Umgebung nicht beim Starten der App von `UseEnvironment` im Host-Builder außer Kraft gesetzt wird, kann die [Seite mit Ausnahmen für Entwickler](xref:fundamentals/error-handling) durch Festlegen der Umgebungsvariable angezeigt werden, wenn die App ausgeführt wird.

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

Das Festlegen der Umgebungsvariablen für `ASPNETCORE_ENVIRONMENT` wird nur bei der Verwendung von Staging- und Testservern empfohlen, die nicht für das Internet verfügbar gemacht werden. Entfernen Sie nach der Fehlerbehebung die Umgebungsvariable aus der Datei *web.config*. Informationen zum Festlegen von Umgebungsvariablen in der Datei *web.config* finden Sie unter [Untergeordnetes environmentVariables-Element von aspNetCore](xref:host-and-deploy/aspnet-core-module#setting-environment-variables).

## <a name="common-startup-errors"></a>Häufige Startfehler 

Weitere Informationen finden Sie in der [Referenz für häufige ASP.NET Core-Fehler](xref:host-and-deploy/azure-iis-errors-reference). Die meisten der häufig auftretenden Probleme, die den Start von Apps verhindern, werden im Referenzartikel behandelt.

## <a name="slow-or-hanging-app"></a>Langsame oder hängende App

Wenn eine App bei einer Anforderung langsam reagiert oder hängt, rufen Sie eine [Dumpdatei](/visualstudio/debugger/using-dump-files) ab und analysieren diese. Dumpdateien können mit einem der folgenden Tools abgerufen werden:

* [ProcDump](/sysinternals/downloads/procdump)
* [DebugDiag](https://www.microsoft.com/download/details.aspx?id=49924)
* WinDbg: [Herunterladen von Debugtools für Windows](https://developer.microsoft.com/windows/hardware/download-windbg), [Debuggen mit WinDbg](/windows-hardware/drivers/debugger/debugging-using-windbg)

## <a name="remote-debugging"></a>Remotedebuggen

Siehe [Remotedebuggen von ASP.NET Core auf einem IIS-Remotecomputer in Visual Studio 2017](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer) in der Dokumentation zu Visual Studio.

## <a name="application-insights"></a>Application Insights

[Application Insights](/azure/application-insights/) stellt Telemetriedaten von Apps bereit, die von IIS gehostet werden, einschließlich Features für die Fehlerprotokollierung und die Berichterstellung. Application Insights kann nur Fehler melden, die nach dem Start der App auftreten, wenn die Protokollierungsfeatures der App verfügbar werden. Weitere Informationen finden Sie unter [Application Insights for ASP.NET Core (Application Insights für ASP.NET Core)](/azure/application-insights/app-insights-asp-net-core).

## <a name="additional-troubleshooting-advice"></a>Weitere Hinweise zur Problembehandlung

Manchmal schlägt eine funktionsfähige App direkt nach der Durchführung eines Upgrades des .NET Core SDK auf dem Entwicklungscomputer oder der Paketversionen in der App fehl. In einigen Fällen können inkohärente Pakete eine App beschädigen, wenn größere Upgrades durchgeführt werden. Die meisten dieser Probleme können durch Befolgung der folgenden Anweisungen behoben werden:

1. Löschen Sie die Ordner *bin* und *obj*.
1. Löschen Sie die Paketcaches unter *%UserProfile%\\.nuget\\packages* und *%LocalAppData%\\Nuget\\v3-cache*.
1. Stellen Sie das Projekt wieder her und erstellen Sie es neu.
1. Überprüfen Sie, ob die vorherige Bereitstellung auf dem Server vollständig gelöscht wurde, bevor Sie die App erneut bereitstellen.

> [!TIP]
> Sie können Paketcaches ganz einfach löschen, indem Sie `dotnet nuget locals all --clear` über eine Eingabeaufforderung ausführen.
> 
> Darüber hinaus können Paketcaches mit dem Tool [nuget.exe](https://www.nuget.org/downloads) und durch Ausführung des Befehls `nuget locals all -clear` gelöscht werden. *nuget.exe* ist wird unter dem Windows Desktop-Betriebssystem nicht gebündelt installiert und muss separat von der [NuGet-Website](https://www.nuget.org/downloads) abgerufen werden.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Einführung in die Fehlerbehandlung in ASP.NET Core](xref:fundamentals/error-handling)
* [Referenz zu häufigen Fehlern bei Azure App Service und IIS mit ASP.NET Core](xref:host-and-deploy/azure-iis-errors-reference)
* [Konfigurationsreferenz für das ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module)
* [Problembehandlung bei ASP.NET Core in Azure App Service](xref:host-and-deploy/azure-apps/troubleshoot)

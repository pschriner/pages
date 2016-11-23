# TYPO3 Grundlagen

## Grundstruktur einer TYPO3 Installation

    typo3conf
        ext
            dm_...
            realurl
        LocalConfiguration.php
        AdditionalConfiguration.php
        PackageStates.php
    typo3
        sysext
            core
            backend
            ...
    *vendor*
        ...

* Die `LocalConfiguration.php` wird vom InstallTool geschrieben
* `AdditionalConfiguration.php` wird nach der `LocalConfiguration.php` geladen (erlaubt es, Werte zu überschreiben)
* `Packagestates.php` ist für vorhandene und geladene Extensions zuständig

Im `vendor` Ordner liegen Packete, die über Composer geladen wurden. Es ist sehr ungewöhnlich in einer nicht-Composer-Installation (unserer) einen `vendor` Ordner zu haben.

## Extensions

    ext_emconf.php

&nbsp;&nbsp;&nbsp;&nbsp;Abhängigkeiten zu anderen Extensions (beeinflusst die Ladereihenfolge), Version, Metadate

    ext_tables.php

&nbsp;&nbsp;&nbsp;&nbsp;"Backend", TCA, Plugins fürs Backend

    ext_tables.sql

&nbsp;&nbsp;&nbsp;&nbsp;Ein SQL Parser baut anhand der Abhängigkeiten von Extensions ein Ziel-Gesamtschema und vergleicht das mit dem gefundenen. Im Installtool gibt es dann eine Analyse die das Hinzufügen // Ändern des Schemas erlaubt.

    ext_localconf.php

&nbsp;&nbsp;&nbsp;&nbsp;Plugins fürs Frontend, Konfiguration

    ext_conf_template.txt

&nbsp;&nbsp;&nbsp;&nbsp;Konfiguration für die Extension, gilt für die gesamte Extension

    ext_typoscript_setup.txt

&nbsp;&nbsp;&nbsp;&nbsp;Typoscript-Konfiguration, die überall gilt (*wird nicht gecached!*)

    Classes
        ...

&nbsp;&nbsp;&nbsp;&nbsp;Hier liegen die Klassen (PSR4)

    Documentation

&nbsp;&nbsp;&nbsp;&nbsp;Früher war hier oft ein sxw, heute ist *reStructuredText* Standard

    Tests

&nbsp;&nbsp;&nbsp;&nbsp;Unit oder Komponententests

    Configuration
        TCA

Die Dateien hier werden gecached. Gute Konvention (und unterstützt) wird es, die Dateien hier mit ###TABELLEN_NAME###.php hinzulegen und nur einen Array zu returnen

        FlexForms

FlexForms sind so etwas wie Mini-TCAs in XML Form für einzelne Plugins. Sie erlauben eine Plugin-spezifische für den Redakteur pflegbare Konfiguration. Da dabei XML in der Datenbank gespeichert wird ist es aber auch eine gute Fehlerquelle.

        (realurl)
        (TypoScript)
    Migration
        Code
            ClassAliasMap.php
    Resources
        Public
            CSS
            JS
            ...

Der `Public` Order und alle darunter werden genutzt um zugängliche Dateien anzubieten

        Private
            Templates
            Layouts
            Partials

Der `Private` Ordner und alle darunter sind nicht für die direkte Anzeige im Frontend bestimmt. Richtig konfiguriert verhindert dies schon die `.htaccess`.

TYPO3 Extensions folgen heutzutage dem Klassen-Schema PSR4

→ products

In Zukunft: Für Composer-Classloading (sinnvoll ab TYPO3 7.6) braucht man eine composer.yml. Diese übernimmt dann das Management der Abhängigkeiten & lässt sich mit Composer für das Deployment nutzen.

## Konzepte

* TYPO3 geht immer von einer Baumstruktur aus (die durch die Tabelle pages festgelegt wird)
* Alle Inhaltselemente (*tt_content*) liegen innerhalb von Seiten
* Alle Datensätze sollten innerhalb von Seiten liegen (storagePid)
* [Versionierung](http://www.schmutt.de/303/eigene-extension-mit-workspace-versionierung/) und Lokalisierung (z.Z. meistens über Overlay)

### Sicherheit

* Im Backend kennt TYPO3 ein Berechtigungskonzept ähnlich zu Unix (User, Group, Any)
* In einer "normalen" TYPO3 Installation sollte es einem Redakteur (nicht-Admin) nicht möglich sein, Sicherheitsprobleme zu verursachen (z.B. XSS)
** Für TYPO3 gilt eine Extension // ein Feature als unsicher, wenn ein Redakteur doch für Sicherheitsprobleme sorgen kann
** Es gibt ein Security-Team und regelmäßig audits von Extensions; Einen klaren Prozess für Sicherheitslücken

### Classloading

* PSR4
  * Besonderheit: Namespace `TYPO3\CMS`
  * idR in Extensions: `VendorName\AaaAaa\B\C` entspricht dann der Datei `typo3conf/ext/aaa_aaa/B/C.php`
* In TYPO3 7.6+ kann man das über einen Eintrag in der `ext_emconf.php' oder der `composer.json` beieinflussen [Helhum](http://insight.helhum.io/post/130876393595/how-to-configure-class-loading-for-extensions-in). Für ältere Versionen gibt es das Konzept `ext_autoload.php`
* [Singletons](https://docs.typo3.org/typo3cms/CodingGuidelinesReference/CodingBestPractices/Singletons/Index.html): `\TYPO3\CMS\Core\SingletonInterface`
* `\TYPO3\CMS\Core\Utility\GeneralUtility::makeInstance()`
* `\TYPO3\CMS\Extbase\ObjectManager->get(BlaBla::class)`
* `@inject` bzw `injectX`
  * Dependency-Injection (nur im Extbase-Kontext)
* Composer

[Helhum](http://insight.helhum.io/post/130812561790/changes-in-class-loading-in-typo3-7lts)

### [Scheduler](https://docs.typo3.org/typo3cms/extensions/scheduler/Installation/SchedulerShellScript/Index.html)

* Wird im *CLI* Kontext ausgeführt (der ist "ähnlich" wie der Backend-Kontext)
* Es macht einen großen Unterschied, ob man einen Task "von Hand" anstößt oder von der Kommandozeile (typo3/cli_dispatch.phpsh scheduler)
* Es gibt "Tasks" und "Commands"
 * "Commands" werden vom Extbase-Task ausgeführt. Sie bieten keine Validierung zur Konfigurationszeit, lassen sich aber auch auf der Kommandozeile ausführen

→ market

### [FAL](https://docs.typo3.org/typo3cms/FileAbstractionLayerReference/)

* Abstraktion des Dateisystems
* Erlaubt es, an Dateien Metadaten zu hängen
* Jede Datei, die innerhalb eines *Storage* ist, bekommt eine Repräsentation in der Datenbank
* Es gibt auch S3 Provider
* Naiver Indexer
* Varianten von Quelldateien
  * *Processing Instructions* - z.B. Verkleinern, Schwarz-Weiß, Wasserzeichen

### [TCA](https://docs.typo3.org/typo3cms/TCAReference/)

Bestimmt wie das Backend Datensätze darstellt. Ermöglicht es TYPO3, Relationen zwischen Tabellen zu verstehen. Bei Extbase wird es auch für das Mapping auf Objekte benutzt

### [typolink](https://docs.typo3.org/typo3cms/TyposcriptReference/Functions/Typolink/Index.html)

Zentrale Stelle, um Links fürs Frontend zu generieren
* funktioniert auch nur im Frontend zuverlässig
* in TYPO3-RTE

### [Hooks](https://docs.typo3.org/typo3cms/CoreApiReference/ApiOverview/Hooks/Concept/Index.html)

* Früher über "Hooks" (z.B. Realurl)
* Heute öfters über Signal-Slot (z.B. extbase)

### [XCLASSes](https://docs.typo3.org/typo3cms/CoreApiReference/ApiOverview/Xclasses/Index.html)

* Früher über XClass-Konvention am Ende einer PHP Datei (z.B. *Realurl*)
* Heute über den `ObjectManager`

```php
$GLOBALS['TYPO3_CONF_VARS']['SYS']['Objects']['TYPO3\\CMS\\Core\\Resource\\ResourceStorage'] = array(
    'className' => 'SchrinerP\\FalPerformance\\Resource\\ResourceStorage'
);
```

## Einstiegspunkte

`$GLOBALS['TSFE']` `\TYPO3\CMS\Frontend\Controller\TypoScriptFrontendController`


`$GLOBALS['TYPO3_DB']` `\TYPO3\CMS\Core\Database\DatabaseConnection`

`\TYPO3\CMS\Core\Utility\GeneralUtility`

Sammlung für alls üble. Vorsicht: Die Suche auf docs.typo3.org findet nicht alle Methoden in dieser Klasse

`\TYPO3\CMS\Frontend\ContentObject\ContentObjectRenderer`

Zentrale Stelle für Typoscript

`\TYPO3\CMS\Core\DataHandling\DataHandler` (früher: *TCEmain*)

Eigentlich nur fürs Backend. Fast jede Änderung im Backend läuft hier durch. Sorgt z.B. für Berechtigungschecks, Referenzindex.

### Typoscript

## Resourcen

 1. [API Docs für TYPO3 6.2](http://api.typo3.org/typo3cms/62/html/) [API](https://api.typo3.org)
 2. [Docs](https://docs.typo3.org)
 3. [Slack](https://typo3.slack.com)
 4. [TYPO3 bei Stackoverflow](http://stackoverflow.com/questions/tagged/typo3)

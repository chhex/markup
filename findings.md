## Ziel / Motivation

Analyse und Beurteilung der Differenzen zwischen

1. dem Ibus Service Branch
   [it21_release_9_0_1_rad_2_admin](https://javabuild.apgsga.ch/view/RELEASE%209.0.1%20Admin/)
2. und dem zukünftigen It21Ui Target Branch
   [JAFO-2059-RC2](https://jenkins.apgsga.ch/view/TempMyBatisMigRC2/job/MyBatis_Mig_Rc_Framework_Persistence/)

mit dem Zeil quantitative and qualitative Aussagen zu einer Migration
des Ibus Service Branches machen zu können.

## Summary

### Generator

Der Generator von IT21 Ui generiert ein paar Runtime Metadaten (
"joinStatement", "tableAliasName"), welche mit dem Ibus Service Stand
generiert wird, nicht. Diese Metadaten werden bei der Sql Generierung
verwendet, dort wo via SqlCriteriaExpressionQueryConfig Qcel Dsl
basierte Datenbankabfragen generiert werden.

Interessant ist, dass das Persistence API für die MetaDaten in beiden
Ständen synchron ist.

Der effektive Impact dieser nicht unterstützten MetaDaten, ist noch
nicht völlig klar und Bedarf weitere Analyse.

Aber aktuell, nach einer
ersten Impact-Analyse in Bezug auf das "joinStatement", sieht es so
aus, dass dessen Funktionalität nur via
[com.affichage.persistence.rest.service](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence.rest.service),
angewendet wird. 

Das "tableAliasName" scheint neu gemäss einer Konvention generiert
werden.

Auch sollte um zu einem abschliessende Aussage der Differenzen zu
allenfalls auch die Differenzen der Source Stände des Generators
analysiert werden.


### Persistence Libraries

Die Source Stände der Persistence Libraries differieren, aber einzig der
"Gap" in der Implementation von DomainWerteRuntime zu grösserem Aufwand
führen kann, da diese beide gegenseitig inkompatibel sind und
unterschiedliche Runtime Anwendungsszenarien unterstützten

## Analyse Differenzen

### Persistence Libraries

#### Vergleichs Methode

Ich habe die Module

- [com.affichage.persistence](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence)
- [com.affichage.persistence.db](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence.db)
- [com.affichage.persistence.java.service](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence.java.service)
- [com.affichage.persistence.rest.service](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence.rest.service)
- [com.apgsga.qcel.dsl](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.apgsga.qcel.dsl)

vom Branch (1) lokal ausgecheckt.

Vom Branch (2) das Module
[apg-persistence-framework](https://fisheye.apgsga.ch/browse/~br=JAFO-2059-RC2/JAVA_CVS/apg-persistence-framework)
ausgecheckt, welches die obigen Module als Subprojekte beinhaltet.

Voraussetzung: das command line [GNU Programm diff](https://www.gnu.org/software/diffutils/) ist installiert

`diff --version diff (GNU diffutils) 2.8.1 Copyright (C) 2002 Free
Software Foundation, Inc.`

Beispiel des modules
[com.affichage.persistence](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence),
sind die Source Stände, wie folgt verglichen worden:

Im directory des Modules [com.affichage.persistence](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence):

`diff -qr --exclude=CVS .
~/cvs/persistence_intellj/apg-persistence-framework/persistence >
diff_files.txt `

und

`diff -bur --exclude=CVS .
~/cvs/persistence_intellj/apg-persistence-framework/persistence >
diff_java_files_content.txt `

verglichen.

Im file diff_files.txt sind alle Files aufgeführt, welche differieren.
Im file diff_java_files_content.txt sind die inhaltlichen Differenzen
per File aufgeführt.

#### Findings

##### Zusammenfassung

In einer ersten Analyse sehe ich grob die folgenden Typen von
Differenzen:

1. Implementations-Verbesserungen resp - Änderungen, welche einzig
   gegebenenfalls auf das Laufzeitverhalten Impact haben (Logging, Bug
   fixes, Parametrisierung etc)

2. Inkompatible API Änderungen. Dh der Code eines Verwender Module
   kompiliert nicht und der migriert werden muss

3. Obsolete(r) Module / Code des Ibus Services Standes, welche nicht
   mehr gebraucht werden.

4. Code, der besser anderswo ist

5. Code / Funktionalität des Ibus Standes, die fehlt im It21 Ui Standes

6. Neue Funktionalität des IT21 Ui Standes

7. kosmetische Änderungen: Formatierung, Kommentar etc

Bei (2), kann man weiter zwischen (a) Implementations API's (Abstrakte
Klassen, Methoden, Klassen Hierarchien etc) und (b) client API's
unterscheiden, welche von anderen Modulen verwendet werden.

Einzig (2 a), (4) und vor allem (5) führen zu Aufwand.

##### Dao Persistence (com.affichage.persistence)

Die Differenzen dieses Modules fallen unter alle Typen.

Die Differenzen von (2 a), die (allenfalls) migriert werden müssen
fallen, sind grob: BeanManager, BeamManagerImpl, Context,
ContextProvider, DbLoadedList, QueryCriteria

Funktionalität, die fehlt im It21 Ui Stand (5): UserContextUtil

Funktionalität, die fehlt im It21 Ui Stand (3), die aber auch obsolet
sein kann: ExcelExportParam

Code / Funktionalität, die besser anders wo ist (4): Die ganzen Spring
Java Config Klassen, die nicht in Tests verwendet werden. (Bemerkung: Ok,
es geht hier vorerst einzig um die, die nicht im It21 Ui Stand sind.)


Für eine detaillierte Analyse der Differenzen siehe
[https://fisheye.apgsga.ch/cru/CR-25#CFR-680](https://fisheye.apgsga.ch/cru/CR-25#CFR-680)


##### Db Persistence (com.affichage.persistence.db)


Die Differenzen hier fallen unter (3), dh. der Code ist obsolet. Es gibt
auch kein equivalent mehr auf dem It21 Ui Branch. Dieser Code wurde
verwendet, um die [Embedded Datendank HSQL](http://hsqldb.org) zu
konfigurieren und patchen im Kontext des RAD Projektes, das aber nicht
mehr verwendet resp verfolgt wird.


##### Java Service Persistence (com.affichage.persistence.java.service)

Funktionalität, die stark differiert und nicht kompatible ist:
[DomainWerteRuntime Stand Ibus Services](https://fisheye.apgsga.ch/browse/~br=it21_release_9_0_1_rad_2_admin/JAVA_CVS/com.affichage.persistence.java.service/src/main/java/com/apgsga/forms2java/persistence/dao/DomainWerteRuntime.java?r=1.1.4.2.2.2.6.1.16.1.18.2)
und [DomainWerteRuntime Stand It21 Ui](https://fisheye.apgsga.ch/browse/~br=JAFO-2059-RC2/JAVA_CVS/apg-persistence-framework/persistence-service/src/main/java/com/apgsga/forms2java/persistence/dao/DomainWerteRuntime.java?r=1.1)

##### Rest Service Persistence (com.affichage.persistence.rest.service)

Die Differenzen hier fallen unter (3), dh. der Code ist obsolet. Dieses
Modul ist ein Persistenz API für GWT Clients, dh der Code sollte im
Kontext der Ibus Services sowieso nicht verwendet werden.

##### Qcel (com.apgsga.qcel.dsl)

Ich habe hier keinen Code Vergleich gemacht, da im Consumer module von
[com.affichage.persistence](https://fisheye.apgsga.ch/changelog/~br=che_hudson_mig_analysis_1/JAVA_CVS/com.affichage.persistence/src/main/java/com/apgsga/forms2java/persistence/qcel?max=30&view=all)
keine Differenzen zu sehen sind.


### Generator

#### Vergleichs Methode

Installation der zwei Eclipse Dists, mit dem jeweiligem Generator Stand.
Das Model der folgenden Module werden mit jeweiligem Generator Stand neu
generiert:

1. [com.affichage.it21.vk.adressliste.excel.dao](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ibus/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao?r=1.1.2.35.4.1)
2. [com.apgsga.it21.lo.rep.begleitzettel.dao](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ibus/JAVA_CVS/com.apgsga.it21.lo.rep.begleitzettel.dao)


Die Änderungen des Generates wurde auf den folgenden Branches
eingecheckt:

[che_hudson_mig_analysis_1_gen_ui](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ui/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao) : Für den It21 Ui Generator
[che_hudson_mig_analysis_1_gen_ibus](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ibus/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao) : Für den Ibus Service Generator
Stand

Die Software Stände wurden auf einer Platform, welche ein GNU diff
unterstützt ausgecheckt:

`diff --version diff (GNU diffutils) 2.8.1 Copyright (C) 2002 Free
Software Foundation, Inc.`


und sind am Beispiel des modules
/com.affichage.it21.vk.adressliste.excel.dao wie folgt verglichen worden

Im directory des Modules
/com.affichage.it21.vk.adressliste.excel.dao.ui_gen:

`diff -qr --exclude=CVS --exclude=target .
../com.affichage.it21.vk.adressliste.excel.dao.srv_gen/ > diff_files.txt
`
und

`diff -bur --exclude=CVS --exclude=target .
../com.affichage.it21.vk.adressliste.excel.dao.srv_gen/ >
diff_java_files_context.txt`

verglichen.

Im file diff_files.txt sind alle Files aufgeführt, welche differieren.
Im file diff_java_files_content.txt sind die inhaltlichen Differenzen
per File aufgeführt.


#### Analyse

1. <Entity Namen>DTOComplex.java Generat Es werden vom Ibus Service
   Generator Java Code mit dem Name <Entity Namen>DTOComplex.java
   generiert. Aus meiner Sicht sind diese DTO's um das client seitige
   Laden von Referenzen und Collections zu vermeiden. Dh vom Client
   benötigte Objektgraf wird vollständig in Form dieser Dto's im Server
   geladen. Die Dtos werden dann Clientseitig wieder zu einem
   navigierbaren Objektgraph zusammengesetzt. Dieses Scenario wird kaum
   im Ibus Service Scenario zur Verwendung kommen => zu verifizieren.

2. Differenzen in der Reihenfolge der Attribute, zb
   [MyDsAdresslisteF Stand Ibus Services](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ibus/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao/src/generated/java/com/affichage/it21/vk/adressliste/excel/dao/MyDsAdresslisteF.java?r=1.1.2.8.4.1)
   vs
   [MyDsAdresslisteF Stand Ibus Services](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ui/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao/src/generated/java/com/affichage/it21/vk/adressliste/excel/dao/MyDsAdresslisteF.java?r=1.1.2.8.2.1)
   ,suche nach agId.

3. Differenzen in der Generierung der Laufzeit Metadaten
   DataMetaInfoImpl,
   [Stand Ibus Services](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ibus/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao/src/generated/java/com/affichage/it21/vk/adressliste/excel/dao/server/DaoMetaInfoImpl.java?r=1.1.2.35.4.1)
   vs
   [It21 Ui](https://fisheye.apgsga.ch/browse/~br=che_hudson_mig_analysis_1_gen_ui/JAVA_CVS/com.affichage.it21.vk.adressliste.excel.dao/src/generated/java/com/affichage/it21/vk/adressliste/excel/dao/server/DaoMetaInfoImpl.java?r=1.1.2.35.4.1).
   Hier fehlt im It21 Ui Stand Funktionalität. Zb. suche
   "addJoinStatement", "addTableAliasName"

(1), (2) sind höchstwahrscheinlich unproblematisch

(3) ein potenzielles "No-Go", potenziell fehlende Funktionalität im It21
Ui Stand => zu verifizieren.

#### TableAliasName und JoinStatement

TODO

Beispiele für Code, welcher nicht im It21 Ui Stand generiert wird

`getPersistenceMetaInfo().addTableAliasName(BEAN_DESC, "AWF");
getPersistenceMetaInfo().addJoinStatement( BEAN_DESC.getProperty(
com.affichage.it21.vk.adressliste.excel.dao.AdresslisteWertF.ADRESSLISTE_WERT_ID_F),
" JOIN ADRESSLISTE_WERT_ID_F AWIF ON (AWIF.ADRW_ID = AWF.ADRW_ID) ");`


Im Model :

`	entity AdresslisteWertF (id=(adrwId)) { oneToMany AdresslisteWertIdF
adresslisteWertIdF (joinColumns=ADRW_ID referencedColumns=ADRW_ID )`

bzw

`readOnly entity ReportcallF ( id=(repcId) tableAlias="RC" ) {
		oneToMany ReportparamF parameter (joinColumns = REPC_ID)
		oneToMany RBegleitzettelMainF report (joinColumns = REPC_ID)
	}`

Das API PersistenceMetaInfo ist aber bei beiden Ständen gleich und
unterstützen "joinStatement" und "tableAliasName"

Auch sind beide Stände von
com.apgsga.forms2java.persistence.qcel.CriteriaExpressionSqlGenerator
gleich, who das "joinStatement" konsumiert wird, nicht nur in Kontext
von qcel aber auch in der Verwendung
com.apgsga.forms2java.persistence.SqlCriteriaExpressionQueryConfig via
dem

Dh. der Persistence Libraries sind hier beide auf dem gleichen Stand,
aber der Generator Stand IT21 Ui unterstützt diese Verwendung von QCEL
nicht.

#### Impact Analyse

TODO  Methode beschreiben und deren Findings

## Notizen zu einem Vorschlag für das weitere Vorgehen

TODO

### Persistence

Ausgehend von It21 UI Stand

1. Migration der Service Infrastructure. Kandidaten sind

- com.affichage.it21.service
- com.affichage.it21.service.base
- und allenfalls andere.

2. Iteratives adressieren der Inkompatibiläten im It21 Ui Stand von
   Persistence
3. DomainWerteRuntime "Gap" adressieren
4. Migration der Birt Infrastruktur Libraries

Die genaue Identifikation der zu migrierenden Module von (1) und (4)
bedingt natürlich eine sorgfältige Dependency Analyse

(1) und (4) : Beide Libraries / Frameworks werden dabei gleich auf den
Standard von https://jenkins.apgsga.ch/view/TempMyBatisMigRC2/ migriert
Dies ist auch ein Arbeitspaket, welches relativ ins sich geschlossen ist
und eine der Voraussetzungen

(3) : Wenn moeglich: Ein API mit unterschiedlichen Implementationen.
welche konfiguriert werden koennen und welche die Anforderungen. der
unterschiedlichen Anwendungsscenarien gerecht wird. Quick fix:
unterschiedlicher Namespace fuer die beiden Implementationen, resp.
Methodennamen. It21 Ui Scenario rueckwaerts kompatible.

### Generator

TODO

## Offene Punkte

TODO


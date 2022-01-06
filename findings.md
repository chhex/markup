### Ziel / Motivation

Analyse und Beurteilung der Differenzen zwischen 

- dem Ibus Service Branch(it21_release_9_0_1_rad_2_admin branch of https://javabuild.apgsga.ch/view/RELEASE%209.0.1%20Admin/)
- und dem It21Ui Branch apg_persistence_1_9_x 

mit dem Zeil eine quantitative and qualitative Aussage zu einer  Migration des Ibus Service Branches machen zu koennen .  

### Persistence Libraries

#### Source Staende 

Ich habe die Module  

- com.affichage.persistence
- com.affichage.persistence.db
- com.affichage.persistence.java.service
- com.affichage.persistence.rest.service 
- com.apgsga.qcel.dsl 

vom Branch it21_release_9_0_1_rad_2_admin lokal ausgecheckt. 

Vom Branch JAFO-2059-RC2 das Module apg-persistence-framework ausgecheckt, welches die obigen Module als Subprojekte beinhaltet.  
Dieser Stand ist der zukuenftige target fuer des It21Ui

#### Vergleichs Methode 

Veraussetzung:  Gnu diff installiert: 

diff --version
diff (GNU diffutils) 2.8.1
Copyright (C) 2002 Free Software Foundation, Inc.


Die Source Staende sind wie folgt verglichen worden, am Beispiel des modules  /com.affichage.persistence

Im directory des Modules /com.affichage.persistence: 

diff -qr --exclude=CVS . ~/cvs/persistence_intellj/apg-persistence-framework/persistence > diff_files.txt 

und 

diff -bur --exclude=CVS . ~/cvs/persistence_intellj/apg-persistence-framework/persistence > diff_java_files_content.txt 

verglichen. 

Im file diff_files.txt sind alle Files aufgefuehrt, welche differieren. 
Im file diff_java_files_content.txt sind die inhaltlichen Differenzen per File aufgefuehrt. 


#### Analyse 

##### Zusammenfassung 


In einer ersten Analyse, sehe ich grob die folgenden Typen von Differenzen: 

A Implementations Verbesserungen resp - Aenderungen, welche gegebenenfalls auf das Laufzeit Verhalten Impact haben (Logging, Bug fixes, Parametrisierung etc) 
B Inkompatible API Aenderungen. Dh der Code eines Verwender Module kompiliert nicht und der migriert werden muss
C Obsolete(r) Module / Code des Ibus Services Standes, welche nicht mehr gebraucht werden.
D Code, der besser anderswo ist
E Code / Funktionalitaet, die fehlt im It21 Ui Standes
F Neue Funktionalitaet des IT21 Ui Standes
G Kosmetische Aenderungen : Formatierung, Kommentar etc

Bei B , kann man weiter zwischen (1) internen API (Abstrakte Klassen, Methoden, Klassen Hierarchien etc) und (2) externen API's unterscheiden, welche von anderen Modulen verwendet werden.

Einzig (B), (D) und vor allem (E) fuehren zu Aufwand.

##### Dao Persistence (com.affichage.persistence)

Die Differenzen dieses Modules fallen unter alle Typen. 

Die Differenzen, die (allenfalls) migriert werden muessen und welche unter B (2) fallen, sind grob : 
BeanManager, BeamManagerImpl, Context , ContextProvider, DbLoadedList, QueryCriteria 

Funktionalitaet, die fehlt im It21 Ui Stand: UserContextUtil

Funktionalitaet, die fehlt im It21 Ui Stand, die aber auch obsolet sein kann: ExcelExportParam 

Code / Funktionalitaet, die besser anders wo ist: Die ganzen Spring Java Config Klasse, die nicht in Tests verwendet werden. Ok, geht es vorerst einzig um die, die nicht It21 Ui Stand sind. 


Fuer eine detaillierte Analyse der Differenzen siehe https://fisheye.apgsga.ch/cru/CR-25#CFR-680 respektive https://fisheye.apgsga.ch/cru/CR-25#CFR-680



##### Db Persistence (com.affichage.persistence.db)


Die Differenzen hier fallen unter C, dh. der Code ist obsolete. Es gibt auch kein equivalent mehr auf dem It21 Ui Branch. 
Dieser Code wurde verwendet um die Embedded Datendank HSQL zu konfigurieren und patchen im Kontext des RAD Ansatzes, der aber nicht mehr verwendet resp verfolgt wird. 


##### Java Service Persistence (com.affichage.persistence.java.service)

Fuenktionaliaet, die stark differiert und nicht kompatible ist: DomainWerteRuntime

##### Rest Service Persistence (com.affichage.persistence.rest.service)

Die Differenzen hier fallen unter C, dh. der Code ist obsolete. 
Dieses Module ist ein Persistenz API fuer GWT Clients, dh der Code sollte im Kontext der Ibus Services sowieso nicht verwendet werden. 

##### Qcel (com.apgsga.qcel.dsl)

Ich habe hier keinen Code Vergleich gemacht, da im Consumer module com.affichage.persistence keine Differenzen zu sehen sind. 

##### Zusammenfassung

Der Hauptentscheid bzw Impact ist, wie mit DomainWerteRuntime umgegangen werden soll.

### Generator 

#### Vergleichs Methode

Installation der zwei Eclipse Dists, mit dem jeweiligem Generator Stand. Das Model der folgenden Module werden mit jeweiligen Generator Stand neu generiert:


- com.affichage.it21.vk.adressliste.excel.dao


Die Aenderungen des Generates wurde auf den folgenden Branches eingecheckt:

che_hudson_mig_analysis_1_gen_ui : Fuer den It21 Ui Generator 
che_hudson_mig_analysis_1_gen_ibus : Fuer den Ibus Service Generator Stand 

Die Staende wuerden auf einer Platform, welche ein GNU diff unterstuetzt ausgecheckt: 

diff --version
diff (GNU diffutils) 2.8.1
Copyright (C) 2002 Free Software Foundation, Inc.


Die Source Staende sind wie folgt verglichen worden, am Beispiel des modules  /com.affichage.it21.vk.adressliste.excel.dao

Im directory des Modules /com.affichage.it21.vk.adressliste.excel.dao.ui_gen: 

diff -qr --exclude=CVS --exclude=target .  ../com.affichage.it21.vk.adressliste.excel.dao.srv_gen/ > diff_files.txt

und 

diff -bur --exclude=CVS --exclude=target .  ../com.affichage.it21.vk.adressliste.excel.dao.srv_gen/ > diff_java_files_context.txt

verglichen. 

Im file diff_files.txt sind alle Files aufgefuehrt, welche differieren. 
Im file diff_java_files_content.txt sind die inhaltlichen Differenzen per File aufgefuehrt. 


##### Analyse 

1. <Entity Namen>DTOComplex.java Generat
Es werden vom Ibus Service Generator Java Code mit dem Name <Entity Namen>DTOComplex.java  generiert. Aus meiner Sicht sind diese DTO's um das client seitige Laden von Referenzen und Collection zumeiden. Dh vom Client benoetigte Objektgraph wird vollstaendig in form dieser Dto's im Server geladen. Die Dtos werden dann Clientseitig wieder zu einem navigierbaren Objektgraph zusammengesetzt. 
Dieses Scenario wird kaum im Ibus Service Scenario zur Verwendung kommen => zu verifizieren. 

2. TODO 

### Notizen zu einem Vorschlag fuer das weitere Vorgehen 

Ausgehend von It21 UI Stand

1. Migration der Service Infrastructure. Kandidaten sind 
- com.affichage.it21.service
- com.affichage.it21.service.base 
- und allenfalls andere. 
2. Interatives addressieren der Inkompatibilaeten im It21 Ui Stand von Persistence
3. DomainWerteRuntime addressieren
4. Migration der Birt Infrastruktur Libraries

Die genaue Identifikation der zu migrierenden Module von (1) und (4) bedingt natuerlich eine sorgfaeltige Dependency Analyse

(1) und (4) : Beide Libraries / Frameworks werden dabei gleich auf den Standard von https://jenkins.apgsga.ch/view/TempMyBatisMigRC2/ migriert
Dies ist auch ein Arbeitspaket, welches relativ ins sich geschlossen ist und eine der Voraussetzungen 

(3) : Wenn moeglich: Ein API mit unterschiedlichen Implementationen. welche konfiguriert werden koennen und welche die Anforderungen. der unterschiedlichen Anwendungsscenarien gerecht wird. 
Quick fix: unterschiedlicher Namespace fuer die beiden Implementationen, resp. Methodennamen. It21 Ui Scenario rueckwaerts kompatible.

### Offene Punkte

1. Analyse Generator Staende und Impact

Kommentar : Allenfalls ist eine Source Code Differenzen Analyse der Generator Versionen  zielfuehrender, 
als des Diffs eines Samples der Generator Versionen Outputs, im mindesten einfacher? STB to decide. 
Wenn ja, brauche ich sicher Unterstuetzung von STB. Gegenfalls kann man auch beides machen. 




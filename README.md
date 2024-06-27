# Netzwerkkonzept PoC

## Beispiel: Grundlegendes VM Setup ohne OSPF

Image


## FRR OSPFv2

Die Netzwerkkonfiguration in FRR OSPF aktiviert OSPF auf allen Subnetzen, die im konfigurierten OSPF Subnetz enthalten sind.
Die OSPF Netzwerkkonfiguration `network 192.168.1.0/24 area 0.0.0.0` aktiviert OSPF z.B. auf einem Interface mit der IP 192.168.1.129/25, aber nicht auf einem Interface mit der IP 192.168.1.1/23.

Dies hat folgende drei Konfigurationsoptionen zur Folge:
1. Automatisch werden alle Subnetze der jeweiligen Node z.B. BSP oder CSP per OSPF "advertised" / erreichbar gemacht. Einmalig wird `network 0.0.0.0/0 area 0` konfiguriert.
2. Es wird explizit und 1 zu 1 konfiguriert, welche Subnetze "advertised" werden sollen. Für jedes Subnetz, das mittels OSPF übertragen werden soll, wird eine Konfigurationsregel z.B. `network 192.168.250.1/24 area 0` eingetragen. Dies hat zur Folge, dass mit jedem neuen Subnetz auf der Node analysiert werden muss, ob dieses  Gegebenenfalls muss eine entsprechende Regel konfiguriert werden.
3. Eine Mischung aus 1 und 2. Für machen Subnetze werden Einzelregeln definiert, andere werden unter einer größeren Subnetzregel zusammengefasst.

Konfigurationsaufwände:
| Variante | Vorteile | Nachteile |
| -------- | -------- | --------- |
| 1 | Sehr einfache Konfiguration. Wird ein neues Subnetz auf der Node hinzugefügt, muss die OSPF Konfiguration nicht angepasst werden. | Es können keine Subnetze von der Übertragung ausgenommen werden. |
| 2 | Es können Subnetze von der Übertragung ausgenommen werden. | Komplexere Konfiguration. Mit jedem neuen Subnetz auf der Node muss analysiert werden, ob dieses Subnetz "advertised" werden soll. Mit jedem neuen Subnetz auf der Node muss die OSPF Konfiguration angepasst werden.
| 3 | Es können Subnetze von der Übertragung ausgenommen werden. Potenziell muss nicht mit jedem neuen Subnetz die OSPF Konfiguration angepasst werden. | Sehr komplexe Konfiguration. Bei jedem neuen Subnetz muss analysiert werden, ob dieses "advertised" werden soll. Auch muss geprüft werden, ob das Subnetz bereits in einer OSPF Konfigurationsregel abgebildet ist. Ist dies der Fall, aber das Subnetz soll nicht übertragen werden, muss eine grobere OSPF Konfiguration in mehrere, spezifischere Regeln unterteilt werden. Changes bzgl. eines neuen Subnetzes sind nicht einheitlich.


Debugging Aufwand bei Netzwerkfehlern betrachten.

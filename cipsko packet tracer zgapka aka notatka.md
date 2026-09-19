
### w nawiasie {} sa argumenty komend. w nawiasie **()** sa akceptowalne przez cisco skroty komend. 

## komendy show (jest ich w pizde wiec najwazniejsze ***imo*** )
- **show ip interface brief** **_(sh ip int br)** --- pokazuje konfiguracje portow jak sa kable podlaczone, wlaczone itp

- **show ip protocol** **_(sh ip prot)_** --- pokazuje protokol wlaczony aktualnie przydatne przy sprawadzaniu czy routing dynamiczny jest wlaczony

- **show vlan _(sh vl)_** - pelne wyswietlenie informacji o vlanach na urzadzeniu
	- **show vlan brief _(sh vl vr)_** - skrotowe wyswietlenie informacji o vlanach na urzadzeniu
## wstepna konfiguracja
- **enable** **_(en)_** --- zawsze na poczatku piszesz
- **configure terminal** **_(conf t)_** --- wejscie do konfiguracji terminala routera
- **interface {interface}** **(int {interfejs} )** ***np [int se0/0/1]*** --- wybierasz kabel do konfiguracji
     - **ip address {adres} {maska}** **_(ip add {adres} {maska} )** ***np [ip add 192.168.0.1 255.255.255.0]*** --- adresacja kabla
     - **no shutdown** **(no shut)** --- wlaczenie kabla WAZNE ZAWSZE PISZ

## nawigacja
- **exit** --- cofa o jedno dzialanie w konfiguracji czyli np z konfiguracji kabla cofa na konfiguracje terminala spowrotem
- **end** --- cofa sie na sam poczatek konfiguracji do stanu router#


## routing dynamiczny

## ***routing rip***
- **router rip** **_(ro rip)_** --- wlaczenie routingu rip
	- **version {wersja ripa (najczesciej 2)}** **(ver {wersja})** ***np [version 2]*** --- wybranie wersji ripa
	 - **network {adres}** **_(net {adres}** --- adres sieci ktora dodajemy do ripa
	 - **no auto-summary** **_(no au)** --- wylaczenie automatycznej sumaryzacji (prawie zawsze sie to robi wiec po prostu pod koniec)
	   ### redystrybucja
		- **redistribute ospf {nr ospf} metric {nr(zaleca sie jak najnizszy czyli 1)}** **_(red ospf {nr ospf} met {nr})_** --- redystrybucja ripa do podsieci z routingiem ospf

## ***routing ospf***
- **router ospf {nr procesu od 1 do 65535}** **_(ro ospf {nr} )_** --- wlaczenie ospf, numer jest obojetny byle by routery co maja sie routowac mialy ten sam nr procesu czyli jak dasz na jednym ro ospf 1 to wszedzie tak samo tez
	- **network {adres} {wildcard mask} area {area nr}** **_(net {adres} {wildcard} area {areanr}_** --- dodanie sieci do routingu _(pojecie wildcard mask to po prostu maska typu 255.255.255.0 tylko ze od tylu czyli 0.0.0.255)_
	  ### redystrybucja do ripa
		- **redistribute rip subnets** **__(red rip sub)__** --- redystrybucja ospf do drugiej podsieci z routingiem ripa


## **routing eigrp**
- **router eigrp {nr procesu od 1 do 65535}** **__(ro eigrp {nr})__** --- wlaczenie eigrp
	- **network {adres} {wildcard}** **__(net {adres} {wildcard}__** --- dodanie sieci do routingu
	- **no auto-summary** **__(no au)__** --- wylaczenie automatycznej sumaryzacji



## DHCP
- **ip dhcp excluded-address {adres}** **_(ip dhcp ex {adres})_**  ***__ALBO__*** **ip dhcp ex {adres 1} {adres 2}** --- tworzy to  jeden adres badz pule od jednego adresu do drugiego gdzie nie moze byc zaadresowane przez dhcp
- **ip dhcp pool {nazwa}** **_(ip dhcp p {nazwa}_** --- wlaczenie dhcp i wejscie do konfiguracji
	- **network {adres} {maska}** **_(net {adres} {maska})_** --- wyznaczenie jaka siec ma miec wlaczone dhcp
    - **default-router {ip routera}** **_(def {ip routera})_** --- default gateway czyli adres ip routera ktory bedzie podawany do kazdego urzadzenia w dhcp
    - **dns-server {adres dns}** **_(dns {adres dns})_** --- ustawienie dns do dhcp

# VLAN
- **vlan {nr}** - tworzy vlan na switchu o takim numerze jaki sie podalo, ta komende trzeba wpisac na kazdym switchu ktory ma miec vlany
	- **name {nazwa vlanu dowolna}** - nazwa vlanu, dowolna
### komendy na switchach na danym interfejsie wybranym (np gig0/1)
- **int gig0/1**
	- **switchport mode {tryb} _(sw mo {tryb acc dla access tr dla trunk})_** - zmienienie trybu vlanu dla w tym przypadku gig0/1
	 
	 ### komendy dla access
	- **switchport access vlan {nr} _(sw acc vl {nr})_** - przypisuje tryb access do danego vlana


	 ### komendy dla trunka
	- **switchport trunk allowed vlan {argumenty wymienione z przykladami ponizej} _(sw tr all vl {argumenty})_**
		- **switchport trunk allowed vlan {nr,nr,nr... lub nr-nr}** - NADPISUJE!!! liste vlanow vlanami podanymi po przecinku lub z podanego range
		- **switchport trunk allowed vlan all** - zezwala na wszystkie vlany
		- **switchport trunk allowed vlan add {nr}** - dodaje nowy vlan do listy nie nadpisujac jej
		- **switchport trunk allowed vlan remove {nr}** - usuwa dany vlan z listy
		- **switchport trunk allowed vlan except {nr,nr,nr lub nr-nr}** - zezwala na wszystkie vlany oprocz tych z wymienionych badz danego range



https://nsrc.org/workshops/2009/summer/presentations/day3/subnetting.pdf

![[Subnetting chart.png]]
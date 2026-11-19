
zarzadzanie uzytkownikami i grupami AD - `dsa.msc`

adsi edit - `adsiedit.msc`
- atrybuty do adsi
	- `title` - stanowisko pracy
	- `department` - dzial pracy
	- `company` - firma
	- `description` - opis
	- `telephonenumber` - numer telefonu
	- `sn` - nazwisko
	- `SamAccountname` - login w starszym formacie
	- `userprincipalname` - login w nowszym formacie
	- `givenname` - imie
	- `displayname` - wyswietlane w drzewie imie
	- `physicaldeliveryoffice` - lokalizacja biura
	- `useraccountcontrol` - status konta, ma wiele wartosci, dwie przykladowe `512` to wlaczone kontom `514` wylaczone, jest ich wiecej ale to sa podstawowe dwie ktore warto zapamietac
	- `homedirectory` - katalog domowy uzytkownika

### strategia KGDLU

Strategia KGDLU ułatwia zarządzanie uprawnieniami w domenie:

- **K** (konta użytkowników) trafiają do **G** (grup globalnych wg roli/działu).
    
- **G** trafiają do **DL** (grup lokalnych w domenie).
    
- **DL** dostają **U** (uprawnienia do zasobów).
    

Dzięki temu administrator zarządza dostępem, przypisując użytkowników tylko do grup globalnych.

### dsmod

przykladowe dodanie do czlonka do grupy poprzez dsmod

`dsmod group "CN=gzar,OU=Zarzad,DC=rol,DC=edu,DC=pl" -addmbr "CN=zar1,OU=Zarzad,DC=rol,DC=edu,DC=pl"`

`dsmod group "CN=grupa,DC=rol,DC=edu,DC=pl" -addmbr "CN=uzytkownik,CN=Users,DC=rol,DC=edu,DC=pl"`


CN w pierwszym argumencie - nazwa grupy

`-addmbr` - samowyjasnialne dodaje czlonka

CN w drugim argumencie - nazwa uzytkownika dodawanego do niej
### przykladowy skrypt eksportujacy czlonkow grupy

```
Import-Module ActiveDirectory

Get-ADGroupMember "DL10DevManagersFullAccess" |
    Select-Object Name, SamAccountName, ObjectClass |
    Export-Csv ".\czlonkowie.csv"
```

`import-module activedirectory` - importuje modul do zarzadzania activedirectory
`get-adgroupmember` - pobiera czlonkow danej grupy 
`select-object` - wybiera wybrane dane
- `name` - nazwe
- `SamAccountname` - login konta
- `objectclass` - co to jest
`export-csv` - eksportuje do pliku csv dane

### przykladowy skrypt powershella tworzacy grupe w AD i dodajacy uzytkownika do niej.
```
import-module activedirectory

New-ADgroup -Name "grupa" -SamAccountname "grupa" -groupscope domainlocal -groupcategory security -path "OU=grupy,DC=domena,DC=com"

add-adgroupmember -identity "grupa" -members "uzytkownik"

```
gdzie 
`import-module activedirectory` - importowanie active directory do skryptu powershella
`new-adgroup` - komenda dodajaca nowa grupe AD
- `-name` - nazwa grupy
- `-samaccountname` - stary format loginu grupy
- `-groupscope` - czy grupa ma byc uniwersalna, globalna, badz lokalna w domenie
- `-groupcategory` - czy grupa ma byc grupa zabezpieczen badz dystrybucji
- `-path` - sciezka gdzie bedzie stworzona grupa
`add-adgroupmember` - dodaje czlonka do grupy AD
- `-identity` - nazwa grupy gdzie uzytkownik bedzie dodawany
- `-members` - uzytkownk ktory bedzie dodany do czlonkostwa w grupie

## CSV z CSVDE
przykladowy plik csv tworzacy uzytkownika piotr wojcik z roznymi atrybutami opisanymi wyzej w adsi
```

DN,objectClass,cn,sAMAccountName,userPrincipalName,givenName,sn,displayName,homeDirectory
"CN=Piotr Wojcik,OU=Zarzad,DC=rol,DC=edu,DC=pl",user,"Piotr Wojcik",piotr.wojci,piotr.wojci@rol.edu.pl,Piotr,Wojcik,piotr.wojci,\\\\ROL\\users\\piotr.wojci\\Dok

```

*uwaga* podwojona jest ilosc slashy w homedirectory poniewaz slashe "uciekaja" w powershellu i to temu *zapobiega*

#### uzycie csvde do importu tych danych

`csvde -i -f danyplikwtymprzypadkupiotr.csv`
gdzie 
`-i` - tryb import
`-f` - dany plik po spacji


### automatyzacja tworzenia wielu kont za pomoca csv bez CSVDE

#### przykladowy plik csv
```
SamAccountName,GivenName,Surname,DisplayName,UserPrincipalName,Password
jkowalski,Jan,Kowalski,jkowalski,jkowalski@rol.edu.pl,R0l!Jan2026_Test
akowalska,Anna,Kowalska,akowalska,akowalska@rol.edu.pl,R0l!Anna2026_Test
pnowicki,Piotr,Nowicki,pnowicki,pnowicki@rol.edu.pl,R0l!Piotr2026_Test
```
gdzie argumenty w pierwszej linijce moga byc nazwane jakkolwiek sie to podoba poniewaz beda one i tak tylko zmiennymi w skrypcie powershella.

#### przykladowy skrypt powershella
```
import-module activedirectory

$users = import-csv plikcsv.csv

foreach ($user in $users) {
	$pass = convertto-securestring $user.password -asplaintext -force
	
	new-aduser -name "$($user.Givenname) $($user.surname)" -samaccountname $user.samaccountname -userprincipalname $user.userprincipalname -givenname $user.givenname -surname $user.surname -displayname $user.displayname -path "OU=przykladowajednostka,OU=przykladowadrugajednostka,DC=domena,DC=com" -accountpassword $password -enabled $true -changepasswordatlogon $true
	   
}
```
gdzie
`$users = import-csv plikcsv.csv` - importuje plik csv do zmiennej $users
`foreach ($user in $users) {}` - jest to petla w ktorej ladnie rozdziela uzytkownikow na atrybuty wiec np $user.displayname dalo by display name kazdego uzytkownika z kolejna w pliku csv
`$pass = convertto-securestring $user.password -asplaintext -force` - jest to zmienna na haslo ktora zamienia haslo podane w pliku csv jako string na zaszyfrowane haslo ktore jest dozwolone w AD (w AD nie da sie dac raw stringa jako haslo bo nie pozwoli!)

`new-aduser` - self explanatory dodaje uzytkownika w AD z danymi atrybutami, atrybuty te mamy przypisane w csv do kazdego danego uzytkownika w tym przypadku wiec wpisujemy np argument od nazwy czyli `-givenname` po czym $user.givename i tak z wszystkim pretty much. jedynie rozni sie pare zmiennych gdzie mozna dac $true albo $false aczkolwiek je mozna znalezc juz w helpie normalnie i tez po nich widac ze sa self explanatory, jedyna uwaga to to ze trzeba byc `-accountpassword` pamietac zeby dac zmienna $pass a nie $user.password poniewaz zmienna $pass juz ma zaszyfrowane nasze haslo ktore jest dozwolone w AD a zmienna $user.password juz ma raw string!

## Ldif

#### przykladowy plik Ldif
```
dn: CN=ela zielinska,DC=domena,DC=com
changetype: add
objectclass: user
cn: ela zielniska
sn: zielinska
givenname: ela
displayname: ela zielinska
samaccountname: ezielinska
userprincipalname: ezielinska@domena.com
```

gdzie:
`dn:` - distinguished name czyli sciezka ktora identyfikuje obiekt jakby ta ela miala byc jeszcze w jakies jednostce organizacyjnej trzeba by bylo dac OU=jednostka przed zdefiniowaniem domeny
`changetype:` - typ zmiany jaka chcemy wprowadzic, moze byc `add`, `modify`, `delete`, badz `modrdn`
`objectclass:` - typ obietku, co to jest
`cn:` - common name, nazwa
`sn:` - surname, nazwisko
`givenname:` - imie
`displayname:` - imie wyswietlane
`Samaccountname:` - starszy typ loginu
`userprincipalname:` - nowszy typ loginu

#### ldifde - komenda na zimportowanie pliku ldif

`ldifde -i -f plik.ldif` 
gdzie:
`-i` - import z pliku
`-f` - file, plik
# Azure-Webshop-SQL

## Relational Database Design for Azure SQL

### DBeaver - Adatbázis csatlakozás és létrehozás

<br>

Az alábbiakban egy webshop relációs adatbázis felépítését fogom bemutatni a DBeaver kliens segítségével, amellyel egy MySQL adatbázis kezelőhöz fogok távolról csatlakozni és felépíteni az adatbázis szerkezetét az alábbi szempontokat figyelembe véve:

- *vásárlók*
- *termékek*
- *termékkategóriák*
- *rendelések*
- *rendelési tételek*
- *fizetési információk alapjai* 

A folyamat első lépcsője így nem lehet más, mint az adatbázis kezelőhöz való csatlakozás és az új adatbázis létrehozása.

<table>
  <tr>
    <td><img src="./webshop/New Database connection.jpg" width="350"></td>
    <td><img src="./webshop/Select database.jpg" width="350"></td>
    <td><img src="./webshop/Create new table.jpg" width="400"></td>  
  </tr>
</table>

Miután csatlakoztunk a DBeaver felületén megjelennek a meglévő MySQL adatbázisok. Az új adatbázis létrehozása után meghatározhatjuk a táblák szerkezeti felépítését, a mezőket és az adattípusokat. A táblák közötti logikai kapcsolatokat pedig az elsődleges kulcsok, valamint az idegen kulcsok segítségével definiáljuk. 

Az alábbi kép megerősíti a sikeres csatlakozást, így minden feltétel adott a sikeres adatbázis felépítéshez:

<img src="./webshop/Connected status.jpg" width="750">

---

### 1. Vásárlók tábla létrehozása (`Customers`)

Az előzetes papírra vetett tervezési lépcsőfolyamatokat ideje formába önteni és éles környezetben is kipróbálni. Az SQL szkriptek megírásával sikeresen implementálhatjuk az elképzelésünket és az ***5 tábla*** létrehozásához megírt utasításokat fogom külön-külön bemutatni.

```sql
create table Customers (
Id int Auto_increment primary key,
First_Name VARCHAR(150),
Last_Name VARCHAR(150),
Phone_Number VARCHAR(150),
Email_Address VARCHAR(150));
```

### 2. Termék kategóriák tábla létrehozása (`Product_Category`)

A táblák létrehozásának sorrendjénél figyelembe kellett venni, hogy az idegen kulcsot tartalmazó táblák előtt mindig a hivatkozni kívánt táblát célszerűbb elkészíteni ezzel is segítve, hogy lehetőség szerint elkerüljük az utólagos kiegészítéseket. Ezzel úgymond egy szülő-gyermek kapcsolat alakul ki a táblák között.

```sql
create table Product_Category(
Id int Auto_increment primary key,
Category_Name VARCHAR(150));
```

### 3. Termékek tábla létrehozása (`Products`)

Ebben a táblában a termékek alapinformációin kívül kap helyet egy idegen kulcs is, ami az előző táblához fog kapcsolódni. 

```sql
create table Products (
Id int Auto_increment primary key,
Product_Name VARCHAR(150),
Price int,
Category_Id int,
constraint fk_category
foreign key (Category_Id) references product_category (Id));
```

### 4. Rendelések tábla létrehozása (`Orders`)

A Rendelések tábla a vásárlások adatait fogja tárolni, amelyekhez egy idegen kulcs segítségével jut hozzá, valamint a tábla lhetőséget biztosít arra, hogy a tranzakció során milyen módot szeretnénk használni.  

```sql
create table Orders (
Id int auto_increment primary key,
Customer_Id int,
Payment_Method VARCHAR(150),
constraint fk_customers
foreign key (Customer_Id) references Customers (Id));
```

### 5. Rendelési tételek tábla létrehozása (`Order_Items`)

A Rendelési tételek táblában kerül sor a teljes összesítésre, ahol megvalósul a több-a-többhöz kapcsolat, ugyanis a rendelésünk során egy termék több rendelésben is szerepelhet és egy rendelésben több ugyanolyan termék is előfordulhat, ami a webshopoknál alkalmazott **kosár logikát** valósítja meg. 

```sql
create table Order_Items (
Id int auto_increment primary key,
Orders_Id int,
Product_Id int,
Quantity int,
constraint fk_orders
foreign key (Orders_Id) references Orders (Id),
constraint fk_products
foreign key (Product_Id) references Products (Id));
```
A létrehozott adatbázis szerkezet a webshop alkalmazás alapja lesz és a táblák közötti kapcsolatok elősegítik, hogy egy-egy rendelési folyamat gördülékenyen végbemenjen. Az alábbi diagram segítségével láthatóak azok a relációs kapcsolatok, amik létrejöttek az egyes táblák között. 

<img src="./webshop/Diagram.jpg" width="400">

---

### DBeaver - Indexek létrehozása

Az indexek segítenek abban, hogy egy összetett adatbázisban - akár több százezer vagy milliós nagyságrendű rekord között is - gyorsan megtaláljuk a keresett elemet ezzel elkerülve a hosszabb várakozási időt. A **Primary Key** (`ID`) segítségével gyorsan hozzáférhetünk egy-egy rekordhoz. Ellenben egy index nélküli `Email_Address` mezőnél a válaszidő megugrik, mert a rendszer **Full Table Scan** keresésbe kezd, ami a rekordok egyesével történő vizsgálatát jelenti az első sortól kezdve. Ez nagyobb adatmennyiségnél válhat igazán kellemetlen folyamattá.

Az index lehetőséget biztosít arra, hogy bizonyos oszlopokat megjelölve a keresések villámgyorsan végrehajtódjanak. Ezen adatbázishoz három index felvételét tartottam releváns opciónak, ami hasznos lehet.

*E-mail címre keresés:*
```sql
create index Email_Index
on Customers (Email_Address);
```

*Terméknévre keresés:*
```sql
create index ProductName_Index
on Products (Product_Name);
```

*Kategória típusra keresés:*
```sql
create index CategoryName_Index 
on Product_Category (Category_Name);
```
---

## Webshop adatbázis felépítése Azure SQL Database segítségével

Az adatbázis elkészítése után egy demo alkalmazást szeretnék működtetni néhány mintaadattal, így a költséghatékonyság elvét szem előtt tartva, olyan adatbázist szeretnék üzemeltetni, ami nem a skálázhatóságra, hanem a minimális erőforrásigény kielégítésére lesz kialakítva.

Az SQL Service Tier és redundancia beállításai megoldásokat kínálnak a költséghatékony működtetésre. Ha nem létszükséglet a nagyobb Storage vagy az írási, olvsási sebesség, akkor a Basic DTU beállítás optimális választás lehet, ami mellett csak minimális havi díjjal kell számolni.

### Azure SQL Database létrehozása

Az Azure Portálra belépve a keresőmezőben rákeresünk az **SQL Database** kifejezésre, majd SQL létrehozásával elindíjuk a folyamatot. 
Főbb pontok, amiket érdemes követni a létrehozás során:

- Egyedi `kiszolgálónév`;
- `SQL autentikáció` vagy `SQL és Microsoft Entra autentikáció` beállítása;
- Autentikációhoz `felhasználónév` és `Jelszó` páros;
- Munkakörnyezet: `Fejlesztő`;
- `Nyilvános végpont` alkalmazása, a `tűzfalszabályok engedélyezése`;
- Létrehozás, majd adatbázis létrejötte.

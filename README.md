# Azure-Webshop-SQL

## Relational Database Design for Azure SQL

### DBeaver - Webshop adatbázis tervezése

<br>

Az alábbiakban egy webshop relációs adatbázis felépítését fogom bemutatni a DBeaver kliens segítségével, amellyel egy MySQL adatbázis kezelőhöz fogok távolról csatlakozni és felépíteni az adatbázis szerkezetét az alábbi szempontokat figyelembe véve:

- vásárlók
- termékek
- termékkategóriák
- rendelések
- rendelési tételek
- fizetési információk alapjai 

A folyamat első lépcsője így nem lehet más, mint az adatbázishoz csatlakozás.

<table>
  <tr>
    <td><img src="./webshop/New Database connection.jpg" width="300"></td>
    <td><img src="./webshop/Select database.jpg" width="300"></td>
  </tr>
</table>

A csatlakozást követően a DBeaver-ben megjelennek a MySQL programban korábban létrehozott adatbázisok. Immáron létrehozhatom a sajátomat, amelyben elkészítem majd a táblázatokat, meghatározom a mezőket és adattípusokat. A táblák közötti logikai kapcsolatokat pedig az elsődleges kulcsok, valamint az idegen kulcsok segítségével fogom meghatározni.




# h3-EternalHomework
Tunkeutumistestaus

Metasploitin peruskomentoja ja termejä ovat muun muassa Exploits (haavoittuvuuden hyödyntämiskoodi), Payload (kohteessa suoritettava koodi), Auxiliary (apuohjelmat mm. skannaukseen), Encoders (piilottelukoodit) ja Meterpreter (muistissa toimiva monipuolinen payload).

Metasploitin etuihin perinteisiin manuaalisiin tekniikoihin verrattuna kuuluvat sen avoin lähdekoodi, helppo laajojen verkkojen testaus CIDR-osoitteilla, älykäs payloadien hallinta sekä puhtaammat poistumiset kohteesta ilman sovellusten kaatumista.

Tiedonkeruussa ja tiedonhallinnassa hyödynnetään tietokantoja kuten PostgreSQL- ja workspace-toimintoa erottelemaan eri projektien tiedot toisistaan.

Skannauksessa käytetään Nmapia (db_nmap) palveluiden tunnistamiseen ja haavoittuvuuksien löytämiseen.

Hyökkäysvaiheessa haavoittuvuus varmistetaan Metasploitin moduleilla ja kohteeseen murtaudutaan, minkä jälkeen komentorivi voidaan päivittää vakaammaksi Meterpreter-istunnoksi.

Jälkikäyttövaiheessa (post-exploitation) hyödynnetään prosessien migrointia piiloutumiseen, verkon reititystä (autoroute) liikenteen ohjaamiseen, incognito-lisäosaa käyttäjätunnisteiden (tokens) varastamiseen sekä mimikatz/kiwi-työkaluja selväkielisten salasanojen ja tiivisteiden dumppaamiseen.

Esimerkkitapauksessa kompromisoidun koneen kautta onnistuttiin etenemään ja saamaan pääsy verkon toisella alueella sijaitsevaan Domain Controlleriin (pivoting).

_________________________________________________________________________________________________________________________________________________________________

a) Mitä 'nmap -sn' tekee? Älä arvaa, vaan perustele lähteillä. Mistä tiedät, että käyttämäsi lähde on luotettava?

Ping Scan / No-port Scan. Se tekee verkon laitetunnistusta eli etsii verkossa olevia aktiivisia koneita ilman, että se skannaa niiden portteja.
Lähteenä luotan tässä MAN sivuja nmpaille. Se lähettää verkkoon erilaisia probe-paketteja. ICMP echo requestit, TCP SYN/ACK -paketteja tai ARP-kyselyitä riippuen ollaanko samassa aliverkossa tarkistaakseen, mitkä IP-osoitteet ovat elossa ja vastaavat.
_________________________________________________________________________________________________________________________________________________________________

b) Tallenna porttiskannauksen tuloksia Metasploitin tietokantoihin. Skannaa niin, että Metasploitable tulee mukaan. Kannattaa ottaa mukaan ainakin versioskannaus -sV (joka on banner grabbing plus).

<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/d55bd1d2-7400-4339-90fc-c38b38ca43de" />

Kohteiden tiedustelu ja tallennus suoraan tietokantaan (db_nmap -sV):

Komento käynnisti Nmap-skannauksen kohteeseen (192.168.128.2) ja käytti versioskannausta (-sV, eli banner grabbing plus), joka selvittää mitä ohjelmitoja ja versioita palvelimella pyörii.

Etuliite db_ ohjasi skannauksen tulokset suoraan Metasploitin sisäiseen PostgreSQL-tietokantaan, jotta tietoja ei tarvitsisi erikseen tuoda tiedostoista.

_________________________________________________________________________________________________________________________________________________________________

c) Tarkastele Metasploitin tietokantoihin tallennettuja tietoja komennoilla "hosts" ja "services". Kokeile suodattaa näitä listoja tai hakea niistä.


<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/582db673-ed48-40ce-9ff9-82c0bda4ba78" />

Tässä tehtiiin tietokannan tietojen tarkastelu 

hosts: Näytti listan tietokantaan tallennetuista isäntäkoneista (IP-osoitteet, käyttöjärjestelmät).

services: Näytti listan löydetyistä avoimista porteista, protokollista ja palveluversioista.

Suodatus ja haku (hosts -S ja services -p 21): näytti miten massiivisesta tietomäärästä voidaan rajata ja hakea  haluttuja kohteita tai tiettyjä portteja (kuten FTP-porttia 21) jatkohyökkäyksiä varten.

_________________________________________________________________________________________________________________________________________________________________

d) Internet famous. Etsi Metasploitablen mukana tulevista hyökkäyksistä (en: exploits; search) sellainen, joka on ollut julkisuudessa.

<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/5e267cde-e8ac-4eec-8957-3b0b3771cd83" />

jos valitaan esimerkiksi vsftpd_234_backdoor -haavoittuvuuden:

Vuonna 2011 vsftpd (Very Secure FTP Daemon) -ohjelmiston version 2.3.4 viralliseen latauslähteeseen murtauduttiin, ja sen lähdekoodiin ujutettiin salainen takaportti.
Jos käyttäjä kirjautui FTP-palvelimelle käyttäjätunnuksella, jonka perässä oli hymynaama (:)), ohjelma avasi salaa kuunteluportin 6200. Tähän porttiin kuka tahansa pystyi yhdistämään suoraan ja saamaan järjestelmästä täydet pääkäyttäjän (root) oikeudet ilman salasanaa.

Miksi "Internet famous"? Tapaus oli valtava uutinen kyberturvallisuusmaailmassa, koska kyseessä oli suositun avoimen lähdekoodin FTP-palvelimen virallinen lähdekoodikompromissi ("supply chain attack"), ja sen helppokäyttöisyys teki siitä yhden historian tunnetuimmista opetus- ja murtoesimerkeistä.

_________________________________________________________________________________________________________________________________________________________________

e) Vertaile nmap:n omaa tiedostoon tallennusta (-oA foo) ja db_nmap:n tallennusta tietokantoihin. Mitkä ovat eri tiedostomuotojen ja Metasploitin tietokannan hyvät puolet?

Perinteisen Nmapin tiedostomuodot (-oA foo) tallentaa skannaustulokset kolmeen eri tiedostomuotoon (.nmap, .gnmap, .xml).

Hyvät puolet:

Siirrettävyys: Tiedostot ovat kevyitä ja helposti siirrettävissä koneelta toiselle 
Yhteensopivuus: XML-muoto voidaan tuoda helposti muihin tietoturvatyökaluihin 
Riippumattomuus: Tulokset säilyvät levyltä luettavassa muodossa, pyöriikö Metasploit tai sen tietokanta taustalla.

Metasploitin tietokanta (db_nmap) tallentaa skannaustulokset suoraan Metasploitin sisäiseen PostgreSQL-tietokantaan.

Hyvät puolet:

Tiedot ovat heti hyödynnettävissä Metasploitin sisällä (esimerkiksi hyökkäysmoduulit voivat käyttää tietokantaan tallennettuja kohdeosoita ja portteja suoraan).
Kaikki skannaukset, isännät (hosts) ja palvelut (services) pysyvät siististi yhdessä paikassa ilman erillistä tiedostojen tuontia tai hallintaa joten keskitetty kanta.
Tietokannasta data on helppo hakea, suodattaa ja tarkastella kohteita suoraan konsolikomennoilla (hosts, services).

_________________________________________________________________________________________________________________________________________________________________

f) Murtaudu Metasploitablen vsftpd-palveluun


<img width="1286" height="843" alt="image" src="https://github.com/user-attachments/assets/fe3f21e3-db17-459c-ac5c-b88dde6b28da" />


_________________________________________________________________________________________________________________________________________________________________

g) Kerää levittäytymisessä (lateral movement) tarvittavaa tietoa metasploitablesta. Analysoi tiedot. Selitä, miten niitä voisi hyödyntää.

Verkon rajapinnat ja muut aliverkot (ip a tai ifconfig) Näyttävät koneen IP-osoitteet. Jos koneella on useampi verkkokortti (esim. toinen sisäverkkoon), se toimii siltana (pivot point) muihin verkkoihin.

ARP-taulu ja reititys (arp -a tai ip neighbor) Paljastaa, minkä muiden laitteiden kanssa tämä kone on äskettäin kommunikoinut samassa aliverkossa.

Aktiiviset yhteydet ja sisäiset palvelut (netstat -antp) Näyttää, mitä palveluita pyörii koneella pelkästään sisäverkon puolella esim. sisäiset tietokannat

Käyttäjät ja SSH-avaimet (cat /etc/passwd, ls -la /home/*/.ssh) komennot tarkistaa järjestelmän käyttäjätilit sekä sen, onko käyttäjille tallennettu SSH-avaimia (id_rsa), joilla kirjaudutaan muille koneille.

Tunnuksien kierrätys (Credential Reuse): Jos konfiguraatiotiedostoista löytyy esim. verkkosovellusten tietoja tai kotihakemistoista käyttäjätunnuksia ja salasanoja, niitä voidaan kokeilla suoraan muihin verkon laitteisiin tai palveluihin, koska ihmiset käyttävät usein samoja salasanoja eri paikoissa.

Mikäli löytää käyttäjältä yksityisen SSH-avaimen, hyökkääjä voi kirjautua sen avulla suoraan muihin järjestelmiin, joissa sama avain on käytössä ilman salasanojen arvuuttelua.

Jos koneella on pääsy sellaiseen sisäverkkoon, johon hyökkääjän Kali-kone ei suoraan yllä, murrettua konetta voidaan käyttää "välityspalvelimena" (Pivotpoint), jonka kautta hyökätään verkon muihi koneisiin.
_________________________________________________________________________________________________________________________________________________________________

h) Murtaudu Metasploitableen jollain toisella tavalla. (Jos tämä kohta on vaikea, voit tarvittaessa turvautua verkosta löytyviin läpikävelyohjeisiin. Merkitse silloin 
raporttiin, missä määrin tarvitsit niitä).

<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/f5976a7c-c60f-4f67-9d69-ab2b2e39a5d4" />

Katselin youtubesta ohjevideoita https://www.youtube.com/watch?v=VmBTZ8xMG14 ja konsultoin samalla geminiä. 

_________________________________________________________________________________________________________________________________________________________________
i) Demonstroi Meterpretrin ominaisuuksia.


<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/dc4789eb-3a39-4c91-aca7-e99ed29f9edc" />
<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/b7823977-4bd6-49c0-82eb-0a3f1d2984eb" />


_________________________________________________________________________________________________________________________________________________________________
j) Tallenna shell-sessio tekstitiedostoon script-työkalulla (script -fa log001.txt) tai tmux:lla.

<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/d1103f15-e760-4fd5-ad9f-4a6a48922bca" />

_________________________________________________________________________________________________________________________________________________________________

k) Pivot point. Laita kaikki harjoituksen tiedostot (script -fa, nmap -oA...) samaan kansioon. Hae sopiva pivot point (sovellus, versio, osoite, MAC-numero) 'grep -r' -komennolla. Keksi uskottava esimerkkikysymys, johon haet vastausta.

<img width="1392" height="952" alt="image" src="https://github.com/user-attachments/assets/5dc0ec0a-eed6-4d71-9a1e-d0e38776aedf" />

_________________________________________________________________________________________________________________________________________________________________

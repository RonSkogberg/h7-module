# H7 Oma moduli

#### Tekijä: Ron Skogberg (13.5.2024)

## Johdanto 

![14](https://github.com/RonSkogberg/h7-module/assets/148875466/409affd6-5bd7-422e-9912-66b2e88a404d)

Hei! Otin modulini tavoitteeksi luoda kuvitellun yrityksen työntekijöille helpon tavan saada tietoa työpaikan ulkopuolella vellovasta säästä. Yrityksen johto on huomannut, ettei sen työntekijät ulkoile tarpeeksi tauoillaan ja tällä ratkaisulla johto yrittää aktivoita heitä enemmän siihen. Yllä oleva kuva toimii esimakuna siitä, mitä on luvassa.

Modulissa käydään läpi Vagrant-virtuaalikoneiden luonti, Saltin master-/minion arkkitehtuurin luonti sekä käyttämieni ratkaisujen selittäminen sekä testaus. Käytän Windows 10 käyttöjärjestelmää ja Windows PowerShelliä, mutta myös Vagrant SSH-yhteyden kautta kolmea Debian-pohjaista virtuaalikonetta. Modulini on julkaistu GNU General Public License v3 alaisilla oikeuksilla ja siinä on sovellettu Tero Karvisen antamaa tehtävänantoa "h7 Oma miniprojekti" hänen Infra as Code -sivultaan (Karvinen 2024). 

Ennen varsinaista infraa koodina, valmistellaan käyttämämme toimintaympäristö.

## Ympäristön valmistelu

Aloitan modulissa käytettävän ympäristön valmistelun luomalla Vagrantilla kolme virtuaalikonetta. Yksi koneista toimii Salt-masterina (heikkihelppari) kahdelle Salt-minionille (tiinatoimari & raimoraksaukko). Hyödynnän ympäristön luonnissa modifoitua versiota Tero Karvisen Vagrantfile-pohjasta (Karvinen 2023) sekä omaa kotitehtäväraporttiani koskien virtuaalikoneiden luontia ja Saltin herra-orja -arkkitehtuurin käyttöönottoa (Skogberg 2024).

### Virtuaalikoneiden luonti Vagrantilla

![0  VagrantFile f](https://github.com/RonSkogberg/h7-module/assets/148875466/102196f4-11d1-40d2-88a6-3a434403927f)

Ollessani Windows PowerShellissä samassa hakemistossa, jossa Vagrantfile sijaitsee, pystyn ```vagrant up``` komennolla luomaan Vagrantfilen määritysten mukaiset virtuaalikoneet. Luotuani koneet otan niihin ssh-yhdeyden ```vagrant ssh <hostname>``` komennolla hallitakseni niitä.

### Salt-herra/-orja -arkkitehtuurin luonti

Ennen Saltin herra-orja -ympäristön luontia päivitän yksitellen kaikkien koneiden pakettivarastot sekä asennan saatavilla olevat päivitykset ```$ sudo apt-get update && sudo apt-get dist-upgrade -y``` komennolla. Tästä en ottanut erillistä kuvaa, sillä toimenpide on tuttu edellisistä tehtävistä ja must-do itselleni aina kun käynnistän virtuaalikoneen pitkän aikavälin jälkeen.

Asennan Salt-masterin heikkihelpparin virtuaalikoneelle käyttäen ```$ sudo apt-get -y install salt-master``` komentoa. 

![1](https://github.com/RonSkogberg/h7-module/assets/148875466/4ca79f12-c3b8-4183-96a4-fe843b1fe67a)

Salt-minionin asennus suoritetaan virtuaalikoneille tiinatoimari ja raunoraksaukko komennolla ```$ sudo apt-get -y install salt-minion```. Seuraavassa kuvassa toimenpide on tehty pelkästään tiinatoimarille, mutta suoritin saman komennon myös raimoraksaukon koneella.

![2](https://github.com/RonSkogberg/h7-module/assets/148875466/d8053f6e-d487-455b-9960-19279a9e161a)

Lisätäkseni minioneiden avaimet masterille muokkaan vuorollaan molemmilla minion-koneilla niiden minion-konfiguraatiotiedostoja komennolla ```$ sudoedit /etc/salt/minion```. Lisään tiedostoon masterin ip-osoitteen sekä minionin tunniste-ID:n. Kuten edellisessä kohdassa, suoritan tämän tiinatoimari-koneen lisäksi raimoraksaukko-koneella (käyttäen raimoraksaukko ID:tä.

![3](https://github.com/RonSkogberg/h7-module/assets/148875466/19cb592d-a0a0-483f-9b3e-7bf0c5608e6c)

Viimeistelläkseni Saltin herra-orja -arkkitehtuurin syntymisen, siirryn master-koneelle (heikkihelppari) hyväksyäkseni minionit. Käyttäen komentoa ```$ sudo salt-key -A``` hyväksyn molempien minioneiden avaimet osaksi masteriani ja täten luon master-minion-yhdeyden. Lopuksi testaan vielä ```$ sudo salt '*' cmd.run whoami``` komennolla, että salt-master ja salt-minionit kommunikoivat onnistuneesti. Molemmat minionit vastasivat masterin pyyntöön, joten yhteydet voidaan todeta toimiviksi.

![4](https://github.com/RonSkogberg/h7-module/assets/148875466/661c977d-3951-4d2e-b35b-784784f4a2ff)

Luotuani kolme virtuaalikonetta ja määriteltyäni niiden välille Saltin herra-orja arkkitehtuurin, pääsen aloittamaan varsinaisen modulin teon.

## Varsinainen moduli

Alkuperäiset suunnitelmani modulin suhteen olivat aika kunnianhimoiset. Tiesin haluavani hakea säätietoja internetistä ja tuoda ne tavalla tai toisella minion-koneille. Halusin alunperin ajastaa jonkinlaisen scriptin, joka suoratulostaisi Salt-masterin kautta päivän sään minioneille. Tein taustatutkimusta aiheesta ja aika nopeasti kävi ilmi, että tällainen ratkaisu on vaikea toteuttaa, ellei mahdotonta. Palasin takaisin suunnittelupöydälle ja aloin suunnittelemaan jotain maltillisempaa.

Päädyin lopulta lähestymistapaan, jossa Salt-minion kopio script-tiedoston Salt-masterilta ja suorittaa sen sisältämän säänhakukomennot. Jos minionilla on jo kyseisen tiedosto, päivittää se sen sisällön vastaamaan masterin vastaavaa. Näin ollen molempien osapuolien script pysyy aina ajan tasalla, vaikka sen sisältö muuttuisi.

Pienen googlettelun jälkeen löysin muutamia säätietoja tarjoavia API-palveluita, mutta useimpiin niistä tuli tehdä tunnukset ja joihinkin niistä olisi pitänyt lisätä ilmaisversiossakin luottokorttitiedot (köh, Weatherstack, köh). Hakujen yhteydessä törmäsin Chubin nimimerkillä toimivaan GitHub-käyttäjään ja hänen luomaansa wttr.in -nimiseen komentokehoteelle suunnattuun sääpalveluunsa. Palvelu hakee säätietoja hänen omalta sivustoltaan (https://wttr.in/), josta ne tulostuu curlilla komentokehotteeseen (Chubin 2024). Se, mistä Chubin itse saa säätietonsa jäi hieman epäselväksi, mutta epäilen hänen saavan ne jostain julksisesta API-palvelusta. Päätin lähteä kokeilemaan Chubinin palvelua, mikä vaati curlin asennuksen sekä master-, että minion-koneille käyttäen komentoa ```$ sudo apt-get install curl```. Eiköhän lähdetä testaamaan!

### wttr.in testaus

Testauksen suoritin master-koneella ```$ curl wttr.in/Helsinki``` komennolla, jonka poimin Chubin GitHub-projektin kuvauksesta (Chubin 2024). Tulostus oli vaikuttava, mutta tarvitsin vain tämän päivän säätulosteen. Pienen selvittelyn jälkeen opin wttr.in -palvelun ominaisuuksista sen, että lisäämällä "?1" komennon perään pystyn rajoittamaan tulostuksen laajuutta tämän päivän säähän. Suoritin ```$ curl wttr.in/Helsinki?1``` komennon, jonka lopputulos vastasi odotuksiani. Uskon pystyväni hyödyntävän tätä projektissani!

![5](https://github.com/RonSkogberg/h7-module/assets/148875466/00763570-6e01-49ec-a031-c48ef48d515b)

### Scriptistä tulosteeksi

Lähdin suorittamaan tehtävää luomalla Salt-masterkoneelle (heikkihelppari) yksinkertaisen script-tiedoston, jonka sisältö koostuu yksinkertaisesta ```curl wttr.in/Helsinki?1``` komennosta. Loin tämän ```$ sudoedit /srv/salt/script.sh``` komennolla.

![6](https://github.com/RonSkogberg/h7-module/assets/148875466/31984eba-4f1a-4994-bde8-ea348d04af88)

Tämän jälkeen luon Saltin sls-tilatiedoston samaan /srv/salt -hakemistoon ```$ sudoedit /srv/salt/run_script.sls``` komennolla. Määrittelen tiedoston luomaan file.managed tilafunktiolla script.sh tiedoston pohjalta kopion minionille, ja lopuksi suorittamaan sen. "name" ilmaisee kopion sijainnin/nimen, "source" lähdedokumentin sijainnin ja "mode" script-tiedoston suoritusoikeudet. Tämä on sitä infraa koodina!

![7](https://github.com/RonSkogberg/h7-module/assets/148875466/96c7ee87-5a33-404f-aa3c-e9a6277296ea)

Suoritettuani ```$ sudo salt-call state.apply run_script``` komennon Salt-minionilla (tiinatoimari), ilmoittaa tulostuksen kopioineen script.sh tiedoston /tmp -sijaintiin ja seuraava osa kertoi suorittaneensa kyseisen scriptin käyttäen cmd.run tilafunktiota. Tulostuksen lopputulos ei yllätyksekseni kuitenkaan vastannut aiemmin suorittamaani ```curl wttr.in/Helsinki?1``` komennon tulosta. Tuloste on aika hirveää luettavaa eikä mene meikäläisen laadunvalvonnan läpi tällaisenaan.

![8](https://github.com/RonSkogberg/h7-module/assets/148875466/bc356688-8340-4d0a-90fb-f8976185b9d6)

### Scriptin viilausta

Tutkittuani tulostusta hetken, päättelin aika nopeasti, että ongelma saattaa liittyä jotenkin tulostuksen muotoiluun. Googlettamalla löysin toisen henkilön kamppailevan samankaltaisen ongelman parissa.  Stack Overflown foorumikeskustelussa meustrus-käyttäjä tarjosi putkituksella jatkettavaa ratkaisua (meustrus 2018). Meustruksen kertoma ``` | sed 's/\x1B[@A-Z\\\]^_]\|\x1B\[[0-9:;<=>?]*[-!"#$%&'"'"'()*+,.\/]*[][\\@A-Z^_`a-z{|}~]//g'``` on melkoinen merkkihirviö, enkä täysin vieläkään ymmärtänyt mitä sen jokainen osa tarkkaan ottaen tekee, mutta näin tiivistetysti voinee sanoa, että se poistaa scriptin tulosteen erilaisia muotoiluja ja värejä yksinkertaistaen sen lopputuloksen. Lisään master-koneella kyseisen lisäyksen script.sh tiedostossa ```curl wttr.in/Helsinki?1``` komennon perään. Lisäsin myös "2>/dev/null", jonka tarkoituksena on ohittaa mahdolliset virheilmoitukset.

![9](https://github.com/RonSkogberg/h7-module/assets/148875466/9f5e5e7b-28f7-4f2b-852d-67a5e848e4fd)

Siirryn takaisin minion-koneelle ja suoritan ```$ sudo salt-call state.apply run_script``` uudestaan tarkistaen tulostuksen lopputuloksen. Tulostuksen ensimmäinen osa ilmaisee jälleen script.sh tiedoston päivittyneen ja toinen osa cmd.run curl Helsingin päivän sään. Lopputuloksen muotoilu on korjautunut! Päivän sää oli ihmisenluettavaa eikä kenellekään jäänyt epäselväski, että tänään (12.5.2024) on melko lämmin päivä.

![10](https://github.com/RonSkogberg/h7-module/assets/148875466/31f5e026-cdae-44b9-9595-1b673d4bac71)

Tavoitteeni on toteutunut! Suorittamalla yksinkertaisen salt-call komennon saavat kuvittelemani yrityksen työntekijät tiedon tämän hetkisestä säästä, jolloin he osaavat varautua siihen taukoillessaan.

### Lisäominaisuuksia

Halusin viedä idean hieman pidemmälle pienellä lisäominaisuudella. Sen lisäksi, että tulosteeseen tulee sen hetkinen sää, tulostaa se myös yksinkertaisia ohjeita taukopukeutumiseen. Suoritin tämän lisäämällä tarvittavia osia itse script.sh -tiedostoon masterilla. Asetin ehtorakenteen niin, että kun säätuloste ilmoittaa ulkolämpötilaksi +10 °C, tulostaa scriptinsuoritus sään lisäksi ilmoituksen "Ulkona on lämmin; mene tauolla ulkoilemaan!". Jos ulkolämpötila on alle +10 °C, tulostuu ilmoitus "Ulkona saattaa olla hieman kylmä; pukeudu lämpimästi, kun menet tauolla ulos.". Lisättyjen komentojen tehtävistä on lisätietoja scriptitiedostossa komentojen yhteydessä muistiinpanoina. 

![11](https://github.com/RonSkogberg/h7-module/assets/148875466/4c398883-c4f6-46d9-bc0d-54264f1e9c12)

Tallennettuani script.sh tiedoston masterilla, on aika suorittaa jälleen ```$ sudo salt-call state.apply run_script``` minionilla. Tällä kertaa käytin raimoraksaukon minion-konetta, sillä olin meinannut unohtaa hänet. Suoritin komennon ja tuloste näytti hyvältä. Määritelty lisätieto vastasi ehtorakennetta, mutta halusin testata sen toimivuuden päinvastaisessa kontekstissa.

![12](https://github.com/RonSkogberg/h7-module/assets/148875466/3fe8158a-16e9-4965-af72-a8560e625479)

Tässä osassa en enää ottanut kuvaa script.sh tiedoston muutoksista, mutta kuten seuraavan tulostuksen alusta käy ilmi, korotin lämpötilarajaksi +20 °C, jolloin sitä pienemmät lämpötilat tulostavat kehotuksen ulkona vaanivasta kylmyydestä. Ulkona oli testaushetkellä "vain" +15 °C, mutta tulostus kehotti pukeutumaan lämpimämmin. Testi oli siis onnistunut!

![13](https://github.com/RonSkogberg/h7-module/assets/148875466/8d4fe8ed-fa2b-43cb-b0fc-e5acfda5a7f0)

## Loppuarvio

Olin ihan tyytyväinen lopputulokseen, vaikka alun suunnitelmat eivät saaneetkaan tuulta alleen. Näin jälkikäteen jäin pohtimaan mitä kaikkea muuta oppimaani olisin voinut sisällyttää tähän omaan moduliini. Ehkä palaan jonain päivänä päivittämään tätä ja silloin minulla on tuoreita uusia ideoita, kuinka viilata tätä projektia eteenpäin. Modulia tehdessä oli myös mukava huomata, kuinka hyvin muistin esimerkiksi yksinkertaisen sls-tilatiedoston konfiguroinnin, vaikka osa muista komennoista oli unohduksissa.

Kiitos, kun jaksoit lukea raporttini!

## References:

Chubin, I. 2024. wttr.in. GitHub. Luettavissa: https://github.com/chubin/wttr.in

Karvinen, T. 2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/

Karvinen, T. 2024: Infra as Code 2024. Luettavissa: https://terokarvinen.com/2024/configuration-management-2024-spring

meustrus 2018. Removing colors from output. Stack Overflow. Luettavissa: https://stackoverflow.com/questions/17998978/removing-colors-from-output

Skogberg, R. 2024: H2 Soitto kotiin. GitHub. Luettavissa: https://github.com/RonSkogberg/palvelinten_hallinta_2024/blob/main/h2-soitto-kotiin.md

# H7 Oma moduuli

## Johdanto

## Ympäristön valmistelu

Aloitan modulissa käytettävän ympäristön valmistelun luomalla Vagrantilla kolme virtuaalikonetta. Yksi koneista toimii Salt-masterina (heikkihelppari) kahdelle Salt-minionille (tiinatoimari & raimoraksaukko). Hyödynnän ympäristön luonnissa modifoitua versiota Tero Karvisen Vagrantfile-pohjasta (Karvinen 2023) sekä omaa kotitehtäväraporttiani koskien virtuaalikoneiden luontia ja Saltin herra-orja -arkkitehtuurin käyttöönottoa (Skogberg 2024).

![0  VagrantFile f](https://github.com/RonSkogberg/h7-module/assets/148875466/102196f4-11d1-40d2-88a6-3a434403927f)

Ollessani Windows PowerShellissä samassa hakemistossa, jossa Vagrantfile sijaitsee, pystyn ```vagrant up``` komennolla luomaan Vagrantfilen määritysten mukaiset virtuaalikoneet. Luotuani koneet otan niihin ssh-yhdeyden ```vagrant ssh <hostname>``` komennolla hallitakseni niitä.

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

Alkuperäiset suunnitelmani modulin suhteen olivat aika kunnianhimoiset. Tiesin haluavani hakea säätietoja internetistä ja tuoda ne tavalla tai toisella minion-koneille. 

Päädyin lähestymistapaan, jossa Salt-minion hakee

Pienen googlettelun jälkeen löysin muutamia säätietoja tarjoavia API-palveluita, mutta useimpiin niistä tuli tehdä tunnukset ja joihinkin niistä olisi pitänyt lisätä ilmaisversiossakin luottokorttitiedot (köh, Weatherstack, köh). Hakujen yhteydessä törmäsin Chubin nimimerkillä toimivaan GitHub-käyttäjään ja hänen luomaansa wttr.in -nimiseen komentokehoteelle suunnattuun sääpalveluunsa. Palvelu hakee säätietoja hänen omalta sivustoltaan (https://wttr.in/), josta ne tulostuu curlilla komentokehotteeseen. (Chubin 2024) Se, mistä Chubin itse saa säätietonsa jäi hieman epäselväksi, mutta epäilen hänen saavan ne jostain julksisesta API-palvelusta. Päätin lähteä kokeilemaan Chubinin palvelua, mikä vaati curlin asennuksen sekä master-, että minion-koneille käyttäen komentoa ```$ sudo apt-get install curl```. Eiköhän lähdetä testaamaan!

Testauksen suoritin master-koneella ```$ curl wttr.in/Helsinki``` komennolla, jonka poimin Chubin GitHub-projektin kuvauksesta (Chubin 2024). Tulostus oli vaikuttava, mutta tarvitsin vain tämän päivän säätulosteen. Pienen selvittelyn jälkeen opin wttr.in -palvelun ominaisuuksista sen, että lisäämällä "?1" komennon perään pystyn rajoittamaan tulostuksen laajuutta tämän päivän säähän. Suoritin ```$ curl wttr.in/Helsinki?1``` komennon, jonka lopputulos vastasi odotuksiani. Uskon pystyväni hyödyntävän tätä projektissani!

![5](https://github.com/RonSkogberg/h7-module/assets/148875466/00763570-6e01-49ec-a031-c48ef48d515b)


![6](https://github.com/RonSkogberg/h7-module/assets/148875466/31984eba-4f1a-4994-bde8-ea348d04af88)

![7](https://github.com/RonSkogberg/h7-module/assets/148875466/96c7ee87-5a33-404f-aa3c-e9a6277296ea)

![8](https://github.com/RonSkogberg/h7-module/assets/148875466/bc356688-8340-4d0a-90fb-f8976185b9d6)

![9](https://github.com/RonSkogberg/h7-module/assets/148875466/9f5e5e7b-28f7-4f2b-852d-67a5e848e4fd)

![10](https://github.com/RonSkogberg/h7-module/assets/148875466/31f5e026-cdae-44b9-9595-1b673d4bac71)


## References:

Chubin, I. 2024. wttr.in. GitHub. Luettavissa: https://github.com/chubin/wttr.in
(Chubin 2024)

meustrus 2018. Removing colors from output. Stack Overflow. Luettavissa: https://stackoverflow.com/questions/17998978/removing-colors-from-output
(meustrus 2018)

Karvinen, T. 2023. Salt Vagrant - automatically provision one master and two slaves. Luettavissa: https://terokarvinen.com/2023/salt-vagrant/
(Karvinen 2023)

Karvinen, T. 2024: Infra as Code 2024. Luettavissa: https://terokarvinen.com/2024/configuration-management-2024-spring
(Karvinen 2024)

Skogberg, R. 2024: H2 Soitto kotiin. GitHub. Luettavissa: https://github.com/RonSkogberg/palvelinten_hallinta_2024/blob/main/h2-soitto-kotiin.md

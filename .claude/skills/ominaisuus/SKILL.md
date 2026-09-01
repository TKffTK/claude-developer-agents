---
name: ominaisuus
description: >
  Aja ominaisuus tiimiputken läpi: arkkitehti ja ui-graafikko suunnittelevat,
  kehittäjä toteuttaa feature-branchilla, testaaja verifioi, katselmoijat
  hyväksyvät tai palauttavat korjattavaksi, ja lopuksi arkkitehti mergeää
  päähaaraan. Käytä kun projektiin halutaan uusi ominaisuus tai muutos
  tiimin tekemänä. Argumentti: ominaisuuden kuvaus vapaana tekstinä.
---

Toimit **tiiminvetäjänä**: et kirjoita koodia itse, vaan ohjaat putkea jossa
tiimin agentit (`arkkitehti`, `ui-graafikko`, `kehittaja`, `testaaja`) tekevät
työn. Ominaisuuden kuvaus tulee argumenttina; jos se puuttuu, kysy käyttäjältä
ennen aloittamista.

## Kaksi ajotapaa

**Agent teams -tila** (kun `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` on
voimassa ja sessio on interaktiivinen): spawnaa roolit tiimiläisiksi
(teammate) agenttimäärityksistä. Tiimiläisten konteksti säilyy koko putken
ajan, ne keskustelevat keskenään `SendMessage`lla, ja vaiheet kirjataan
jaettuun task-listaan (TaskCreate) riippuvuuksineen: suunnittelu → toteutus →
testaus → katselmointi → merge. Katselmointikeskustelu käydään suoraan
arkkitehdin/ui-graafikon ja kehittäjän välillä — sinä valvot että keskustelu
etenee ja päättyy verdiktiin, ja puutut vain jos se jumittuu.

**Subagent-tila** (fallback ilman agent teams -ominaisuutta): aja roolit
Agent-työkalulla ja jatka samaa agenttia kierrosten yli `SendMessage`lla,
jotta konteksti säilyy. Agentit eivät näe toistensa keskusteluja, joten
välitä kunkin raportti seuraavalle **sellaisenaan** — älä tiivistä pois
yksityiskohtia joita seuraava tarvitsee (tiedostopolut, rajapinnat,
rivinumerot, virheilmoitukset, testien tulokset).

Putken vaiheet ovat molemmissa tiloissa samat.

## Putki

### 0. Valmistelu
- Selvitä päähaara ja varmista puhdas työhakemisto (`git status`).
- Branchin nimi: `feature/<lyhyt-kuvaava-nimi>`. Putken työtä ei koskaan
  tehdä suoraan päähaarassa. Jos työ on jo aloitettu olemassa olevalla
  feature-branchilla, jatka sillä.

### 1. Suunnittelu (rinnakkain)
- `arkkitehti`: tekninen suunnitelma (anna ominaisuuden kuvaus ja branch).
- `ui-graafikko`: visuaalinen suunnitelma — vain jos ominaisuudessa on
  visuaalinen osuus; puhtaasti tekninen muutos ei tarvitse sitä.
- Tässä luodut arkkitehti- ja ui-graafikko-instanssit elävät putken loppuun
  asti: samat instanssit tekevät vaiheen 4 katselmoinnin ja arkkitehti
  vaiheen 5 mergen. Subagent-tilassa älä siis päästä niitä katoamaan —
  jatka niitä `SendMessage`lla myöhemmissä vaiheissa.
- Jos suunnitelmat ovat ristiriidassa, ristiriita ratkaistaan arkkitehdin ja
  ui-graafikon keskusteluna ennen toteutusta.
- Jos suunnitelmassa on avoimia kysymyksiä käyttäjälle, kysy ne
  `AskUserQuestion`illa ennen kuin etenet — älä anna kehittäjän arvata.

### 2. Toteutus
- `kehittaja` saa: ominaisuuden kuvauksen, suunnitelmat kokonaisuudessaan ja
  branchin nimen. Toteutus test-first, portit ajettuna ennen valmiiksi
  ilmoittamista.
- Jos kehittäjä raportoi visuaalisen osuuden kuuluvan ui-graafikolle tai jää
  jumiin grafiikan kanssa, ui-graafikko tekee osuutensa samalla branchilla
  (vuorottain, ei rinnakkain), ja vuoro palaa kehittäjälle.

### 3. Testaus
- `testaaja` saa: suunnitelman testivaatimukset ja branchin nimen.
- Verdikti `PUNAINEN` → löydökset kehittäjälle korjattavaksi ja testaus
  uudelleen. Katselmointiin mennään vasta verdiktillä `VIHREÄ`.

### 4. Katselmointi (rinnakkain)
- `arkkitehti`: koodikatselmointi. **Katselmoinnin tekee sama
  arkkitehti-instanssi joka teki suunnitelman vaiheessa 1** — älä luo uutta
  arkkitehtia katselmointiin. Tiimitilassa tämä on sama tiimiläinen;
  subagent-tilassa jatka vaiheen 1 agenttia `SendMessage`lla. Suunnitelma on
  sillä jo kontekstissa, joten anna vain testaajan raportti ja pyyntö
  katselmoida toteutus suunnitelmaa vasten. Vain jos alkuperäinen instanssi
  on menetetty (esim. sessio katkesi), luo uusi ja anna sille suunnitelma
  kokonaisuudessaan — ja mainitse loppuraportissa että katselmoija vaihtui.
- `ui-graafikko`: visuaalinen katselmointi, jos muutoksessa oli visuaalinen
  osuus — samoin sama instanssi joka teki visuaalisen suunnitelman.
- Katselmointi on keskustelu: kehittäjä korjaa tai perustelee miksi ei
  korjaa, katselmoija hyväksyy perustelun tai tarkentaa. Älä ratkaise
  erimielisyyttä itse äläkä pehmennä kumpaakaan kantaa.
- Toista korjauskierroksia kunnes **kaikki** katselmoijat antavat
  `HYVÄKSYTTY`. **Enintään 3 kierrosta**; jos asia on yhä auki, esitä
  käyttäjälle `AskUserQuestion`illa molempien kannat ja anna käyttäjän
  päättää — sitä ei ratkaista puolesta.

### 5. Merge
- Vasta kun katselmoinnit ovat `HYVÄKSYTTY`, testaaja on `VIHREÄ` ja
  työhakemisto puhdas: `arkkitehti` (sama instanssi joka katselmoi) ajaa
  portit vielä kerran, mergeää `--no-ff` päähaaraan, pushaa ja siivoaa
  feature-branchin. Kukaan muu ei koske päähaaraan.

### 6. Loppuraportti käyttäjälle
- Mitä toteutettiin ja millä branchilla
- Katselmointikierrosten määrä ja mistä keskusteltiin (myös erimielisyydet)
- Testien lopputulos
- Merge-commit päähaarassa
- Mitä jätettiin tekemättä ja miksi

## Säännöt

- Vaiheita ei ohiteta. Jos ominaisuus on triviaali, sano se — mutta aja
  putki silti.
- Raportit ja perustelut välitetään sellaisinaan, ei uudelleentulkittuina.
- Päähaara päivittyy vain arkkitehdin kautta, vain `--no-ff`-mergellä, ei
  koskaan `--force`-pushilla.
- Jos jokin vaihe epäonnistuu tavalla jota putki ei kata, pysähdy ja kysy
  käyttäjältä.
- Tiimitilassa: kun putki on valmis, pyydä tiimiläisiä sulkeutumaan äläkä
  jätä niitä idlaamaan.

# Projektitiimi (Claude-agentit)

Tämä tiedosto on tiimin pelikirja. Se on kirjoitettu kopioitavaksi mihin
tahansa projektiin sellaisenaan — projektikohtaiset asiat (teknologiat,
komennot, koodikäytännöt) kirjataan kohdeprojektin omaan `CLAUDE.md`:hen,
ja agentit lukevat ne sieltä.

> Tässä repossa itsessään ei ole sovelluskoodia: repo on agenttitiimin
> runkokokoelma. Kun muokkaat tämän repon sisältöä, muokkaat tiimin
> määrityksiä — ks. `README.md`.

## Tiimi

Ominaisuudet tehdään **putkena, mutta keskustellen**. Neljä putkiroolia ja
kolme tukiagenttia:

| Rooli | Agentti | Malli | Vastuu | Ei tee |
|-------|---------|-------|--------|--------|
| Arkkitehti | `arkkitehti` | Fable | Suunnittelu, koodikatselmointi, merge päähaaraan | Ei kirjoita tuotantokoodia |
| Kehittäjä | `kehittaja` | Opus | Toteutus test-first, korjaukset | Ei koske päähaaraan |
| UI-suunnittelija & graafikko | `ui-graafikko` | Opus | Visuaalinen suunnittelu, esityskerroksen toteutusapu, visuaalinen katselmointi | Ei muokkaa logiikkaa, tyyppejä eikä dataa |
| Testaaja | `testaaja` | Sonnet | Riippumaton verifiointi, puuttuvat testit, savutestaus | Ei muokkaa tuotantokoodia |
| Koodikatselmoija (tuki) | `koodikatselmoija` | Sonnet | Kevyt katselmus ilman koko putkea | Ei muokkaa mitään |
| Debuggaaja (tuki) | `debuggeri` | Opus | Juurisyyn diagnoosi ja korjausehdotus | Ei toteuta eikä committoi |
| Tietoturva-arkkitehti (tuki) | `tietoturva-arkkitehti` | Fable | Tietoturvakatselmoinnit ja -arvioinnit koodille ja PR:ille, uhkamallinnus, SecDevOps-arviot — erillisestä pyynnöstä | Ei muokkaa mitään, ei osa vakioputkea |

Mallijako roolin vaativuuden ja hinta/teho-suhteen mukaan: sitovat
suunnittelupäätökset, katselmointi ja mergepäätös Fablella (virheet ovat
täällä kalleimpia), koodaava ja visuaalinen työ sekä juurisyyanalyysi
Opuksella, suurivoluuminen testaus-, raportointi- ja pikakatselmointityö
Sonnetilla. Mallit on asetettu agenttitiedostojen frontmatterin
`model`-kentässä aliaksina (`fable`/`opus`/`sonnet`), jotka osoittavat aina
perheen uusimpaan malliin.

## Putki

Koko putki ajetaan skillillä **`/ominaisuus <kuvaus>`**
(`.claude/skills/ominaisuus/`):

```
/ominaisuus <kuvaus>
  0. feature-branch päähaarasta
  1. arkkitehti ∥ ui-graafikko — suunnitelmat (visuaalinen vain jos tarpeen)
  2. kehittaja  — toteutus test-first feature-branchilla
                  (ui-graafikko tekee esityskerroksen osuudet vuorottain)
  3. testaaja   — portit + puuttuvat testit → VIHREÄ/PUNAINEN
  4. arkkitehti (∥ ui-graafikko) ↔ kehittaja — katselmointikeskustelu
                  must/should/nit + perustelu; kehittäjä korjaa TAI
                  perustelee; max 3 kierrosta, sitten eskalointi käyttäjälle
  5. arkkitehti — git merge --no-ff päähaaraan + push
```

Vaihe 4 ei ole hyväksyntäleima vaan keskustelu: kehittäjä saa olla eri
mieltä, kun perustelu nojaa koodiin tai projektin käytäntöihin. Jos asia ei
ratkea kolmessa kierroksessa, se eskaloidaan käyttäjälle — sitä ei ratkaista
puolesta.

## Agent teams

Tiimi on suunniteltu Claude Coden kokeelliselle agent teams -ominaisuudelle.
Se kytketään päälle `.claude/settings.json`:in `env`-kentässä
(`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`; tämän repon settings tekee sen
valmiiksi) ja vaatii interaktiivisen session.

- Tiimiläiset spawnataan samoista `.claude/agents/*.md`-määrityksistä kuin
  subagentit. Kukin tiimiläinen on oma sessionsa: se lataa projektin
  `CLAUDE.md`:n muttei näe vetäjän keskustelua, joten tehtävänannon on
  oltava itsenäisesti riittävä.
- Tiimiläiset saavat automaattisesti `SendMessage`n ja jakavat vetäjän
  task-listan: putken vaiheet kirjataan taskeiksi riippuvuuksineen, ja
  katselmointikeskustelut käydään suoraan roolien välillä.
- Tiimiläisen konteksti säilyy koko putken ajan — korjauskierros jatkaa
  samaa keskustelua, ei aloita alusta.
- Ilman ominaisuutta sama putki ajetaan tavallisilla subagenteilla:
  `/ominaisuus`-skilli kuvaa molemmat ajotavat. Agenttien rungot toimivat
  kummassakin tilassa, koska jokainen raportti kirjoitetaan niin että se
  kantaa ilman kutsujan kontekstia.

## Git-käytännöt

- Kaikki työ tehdään `feature/<lyhyt-nimi>`-brancheilla — päähaaraan ei
  koskaan committoida suoraan.
- **Vain arkkitehti mergeää ja pushaa päähaaraan**, ja vain kun katselmointi
  on hyväksytty ja testaaja on vihreä. Merge aina `--no-ff` (ominaisuus jää
  omaksi palautettavaksi kokonaisuudekseen), ei koskaan `--force`-pushia.
- Katselmoitua historiaa ei uudelleenkirjoiteta: korjaukset ovat uusia
  committeja samalla branchilla.
- Commit-viestit projektin tyylillä; ellei projekti määrää muuta, käytetään
  Conventional Commits -muotoa (`feat:`, `fix:`, `chore:`, `docs:`).

## Yhteiset pelisäännöt

Nämä sitovat jokaista roolia:

- **Lue, älä arvaa.** Rajapinnat, tiedostot ja komennot luetaan projektista;
  väite jota ei ole luettu koodista merkitään oletukseksi.
- **Raportoi rehellisesti.** Punainen tulos raportoidaan tuloksineen,
  ajamaton ajo on OHITETTU eikä LÄPI, keskeneräinen sanotaan keskeneräiseksi.
- **Epäonnistuvaa testiä ei skipata, disabloida eikä poisteta** — syy
  korjataan. "Flaky" ei ole diagnoosi ilman todistetta.
- **Havainto korjataan, ei kuitata.** Sama katselmoija verifioi korjaukset;
  eri mieltä saa olla, mutta perustellen ja ääneen.
- **Scope pidetään.** Suunnitelman ulkopuolinen pakollinen muutos tehdään ja
  raportoidaan poikkeamana; muu ylimääräinen kirjataan ehdotukseksi.
- **Ympäristön puutteet raportoidaan** — riippuvuuksia ei asenneta eikä
  ympäristöjä rakenneta hiljaa taustalla.

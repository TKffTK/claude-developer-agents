# Claude kehitysagentit

Yleispätevä, suomenkielinen agenttitiimin runko Claude Code -projekteihin:
roolimääritykset (`.claude/agents/`), tiimiputken ajava skilli
(`.claude/skills/ominaisuus/`) ja tiimin pelikirja (`CLAUDE.md`). Rungot on
suunniteltu Claude Coden kokeelliselle **agent teams** -ominaisuudelle
(`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`), mutta ne toimivat myös tavallisina
subagentteina.

Tiimi on koostettu viiden projektin (mm. alko-systembolaget,
webgl-rahapuu, rehellinen_vaalikone) käytössä hioutuneista
agenttimäärityksistä.

## Roolit ja mallit

| Agentti | Rooli | Malli | Miksi tämä malli |
|---------|-------|-------|------------------|
| `arkkitehti` | Suunnittelu, koodikatselmointi, merge (portinvartija) | `fable` | Sitovat suunnittelu- ja mergepäätökset — virheet ovat täällä kalleimpia, ja rooli ajaa vähiten tokeneita suhteessa vaikutukseensa |
| `kehittaja` | Toteutus test-first feature-branchilla | `opus` | Koodauksen työjuhta: paras teho/hinta varsinaiseen toteutukseen |
| `ui-graafikko` | Käyttöliittymäsuunnittelu, grafiikka, visuaalinen katselmointi | `opus` | Visuaalinen suunnittelu ja shader-/tyylitoteutus ovat koodaavaa asiantuntijatyötä |
| `testaaja` | Riippumaton verifiointi, testien täydennys, savutestaus | `sonnet` | Suurivoluuminen ajaminen ja raportointi — kurinalaisuus tulee rungosta, ei mallikoosta |
| `koodikatselmoija` | Kevyt katselmus putken ulkopuolella | `sonnet` | Nopea ja halpa pikatarkistus; raskas katselmointi kuuluu arkkitehdille |
| `debuggeri` | Virheiden juurisyyn diagnoosi | `opus` | Juurisyyanalyysi on päättelytyötä, jossa väärä diagnoosi maksaa kierroksen |
| `tietoturva-arkkitehti` | Tietoturvakatselmoinnit, auditoinnit annettua mallia vasten (esim. ASVS) raportteineen ja tiketteineen, uhkamallinnus, SecDevOps (erillisestä pyynnöstä, ei vakioputkessa) | `fable` | Ohi mennyt haavoittuvuus on tiimin kallein virhe, ja rooli ajetaan harvoin — laatu ratkaisee, volyymi ei |

Aliakset (`fable`/`opus`/`sonnet`) osoittavat aina malliperheen uusimpaan
versioon, joten rungot eivät vanhene mallijulkaisujen myötä.

## Mukana tulevat skillit

- `.claude/skills/ominaisuus/` — tiimiputken ajava skilli (tämän repon omaa
  sisältöä).
- `.claude/skills/frontend-design/` — Anthropicin
  [anthropics/skills](https://github.com/anthropics/skills)-kokoelmasta
  kopioitu skilli (Apache 2.0, lisenssi hakemistossa mukana). Antaa
  `ui-graafikko`-agentille esteettisen suunnan: omaleimainen visuaalinen
  identiteetti, harkittu typografia ja template-oletusten välttäminen.
  Agenttirunko ohjeistaa lukemaan sen ennen suunnittelua; projektin oma
  tyyliohje voittaa ristiriitatilanteessa. Päivitys tehdään kopioimalla
  upstream-versio uudelleen.

## Käyttöönotto projektissa

1. Kopioi `.claude/agents/`, `.claude/skills/` ja halutessasi
   `.claude/settings.json` projektiisi. Kopioi `CLAUDE.md`:n tiimiosuus
   projektin omaan `CLAUDE.md`:hen (poista lainauslaatikko joka koskee vain
   tätä repoa).
2. Kirjoita projektin `CLAUDE.md`:hen projektikohtaiset asiat: teknologiat,
   **porttikomennot** (testit, type-check, lint, build), testien sijainti,
   commit-tyyli ja koodikäytännöt. Agentit on kirjoitettu lukemaan nämä
   projektin `CLAUDE.md`:stä — runkoja ei tarvitse muokata.
3. Agent teams kytkeytyy päälle `settings.json`:in kautta
   (`"env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" }`). Ominaisuus on
   kokeellinen ja vaatii interaktiivisen session; ilman sitä
   `/ominaisuus`-skilli ajaa saman putken tavallisilla subagenteilla.
4. Aja putki: `/ominaisuus <kuvaus>`.

## Suunnitteluperiaatteet

- **Putki, mutta keskustellen**: katselmointi on vuoropuhelu (must/should/nit
  + perustelu), ei hyväksyntäleima. Kehittäjä saa olla eri mieltä
  perustellen; ratkeamaton asia eskaloidaan käyttäjälle kolmen kierroksen
  jälkeen.
- **Yksi portinvartija**: vain arkkitehti mergeää, aina `--no-ff`, ei koskaan
  `--force`. Puuttuva katselmointi ei ole hyväksytty katselmointi.
- **Riippumaton verifiointi**: testaaja ei muokkaa tuotantokoodia eikä
  raportoi vihreäksi mitään mitä ei ajanut.
- **Roolirajat kirjoitettu auki**: jokaisella roolilla on "ei tee" -lista,
  koska rajat pitävät putken rehellisenä paremmin kuin ohjeet.
- **Kaksikäyttöiset rungot**: sama määritys toimii teammate-sessiona
  (pysyvä konteksti, SendMessage, jaettu task-lista) ja subagenttina
  (raportti kirjoitetaan niin että se kantaa ilman kutsujan kontekstia).

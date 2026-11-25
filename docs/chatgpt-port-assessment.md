# Portering av P&ID to Knowledge Graph till ChatGPT och Codex

**Beslutsunderlag 2026-11-25 · Granskad version `7ee3a82`.** Den lokala porten kan använda samma extraktionsschema, grafbyggare, frågeverktyg och exportformat. Arbetet ligger i att installera skillen, ge modellen åtkomst till rätt bildutsnitt och kontrollera att varje valt ritningsblad kommer med. För ChatGPT i webbläsaren tillkommer ett körprov av Python-bibliotek, bildvisning och filutlämning. Lika god ritningstolkning som med Claude återstår att verifiera på verkliga ritningar. [Arbetsflödet vid den granskade versionen](https://github.com/jesaro-bioshare/PID-to-Knowledge-Graph/blob/7ee3a82/skills/pid-to-kg/SKILL.md)

## Vad behöver ändras?

### Lokal Codex

| Fil eller plats | Åtgärd för lokal port | När ändringen är verifierad |
|---|---|---|
| [`skills/pid-to-kg/`](../skills/pid-to-kg/) och [`README.md`](../README.md) | Behåll en originalkatalog. Dokumentera installation av **hela** katalogen i den arbetsytas `.agents/skills/pid-to-kg/` där användaren öppnar ritningarna, alternativt på användarnivå. Installation kan ske genom kopiering eller en länk som Codex kan följa. En klon i en underkatalog gör inte automatiskt skillen tillgänglig från en överordnad arbetsyta. | I en ny session går det att välja `$pid-to-kg` och starta den från en projektmapp med mellanslag, utanför skill-katalogen. Referenser, scheman och skript hittas. [OpenAI om sökvägar och skillformat](https://learn.chatgpt.com/docs/build-skills) |
| [`SKILL.md`](../skills/pid-to-kg/SKILL.md) och ny `skills/pid-to-kg/pixi.toml` med `pixi.lock` för den lokala Pixi-varianten | Ersätt den ovillkorliga instruktionen att köra `pip install` med ett miljöval. För den undersökta Windows-miljön, använd en dedikerad låst Python 3.12-miljö. Ange win-64 för Windows-piloten. Ta beroendena från [`requirements.txt`](../skills/pid-to-kg/scripts/requirements.txt), inklusive RapidOCR för full funktion. Kör `pixi run --locked --manifest-path <miljö>/pixi.toml python <skill>/scripts/<skript>.py …` med citerade sökvägar. Lägg arbetet i en skrivbar projektmapp. Om den installerade skillen är skrivskyddad ska själva Pixi-miljön ligga på en skrivbar plats. | En körning från en ny miljö öppnar en PDF med PDFium, renderar en PNG, kör OCR på en skannad tagg samt skapar och öppnar en DuckDB-fil. Repots villkor installerar OCR endast före Python 3.13. En körning utan OCR ska redovisa bortfallet. [Projektberoenden](../pyproject.toml) |
| [`SKILL.md`](../skills/pid-to-kg/SKILL.md) och [`references/extraction_procedure.md`](../skills/pid-to-kg/references/extraction_procedure.md) | Beskriv hur värden visar en PNG **som bild för modellen**. Öppna först `overview_grid.png` för orientering, därefter varje icke-tom originalruta enligt `tiles.json`. Anteckna undantag som proceduren uttryckligen tillåter för exempelvis en titelruta som säkert kan granskas i översikten. Följ rör över överlappen och granska osäkra taggar, mätuttag och pilar med `crop.py`. Spara `extract/page_NN.json` innan nästa blad. Vid ändrad rotation ska bildrutor, OCR-hänvisningar och positioner granskas på nytt. Korrigera beskrivningen av OCR-bortfall. Textlagret kan fortfarande ge ledtrådar och en ren skanning får då källan `none`. | Modellen kan ange tagg, ventilordning, mätsida och motsvarande ruta från en bild och kan gå tillbaka till rätt utsnitt efter en QA-varning. Ett blad med roterad text provas. [Bildutsnitt](../skills/pid-to-kg/scripts/crop.py) · [Hantering av textlager](../skills/pid-to-kg/scripts/make_hints.py) |
| [`SKILL.md`](../skills/pid-to-kg/SKILL.md), bygg- och slutrapporteringsstegen | Jämför valt sidurval i `manifest.json` med **sidnumren i** extraktionsfilerna före slutrapport. Kräv en fil med rätt nummer per inkluderat blad. Redovisa tomma blad, uteslutna revisioner och andra undantag. Använd en egen arbetskatalog per PDF och konfiguration och bygg med det uttryckliga urvalet. Vid återupptagning måste PDF, urval och rotation stämma. QA-fel kräver ny granskning och bygge. | Utelämna ett valt blad och lägg till en gammal extraktionsfil. Båda avvikelserna ska upptäckas före leverans. Byggaren läser annars de extraktioner som råkar finnas, och QA kan bara kontrollera blad den känner till. Den skriver även resultat när QA ger felkod. [Byggaren](../skills/pid-to-kg/scripts/build_kg.py) · [QA](../skills/pid-to-kg/scripts/pidkg/qa.py) |
| [`scripts/render_pages.py`](../skills/pid-to-kg/scripts/render_pages.py) | Korrigera kommentaren om Claudes nedskalning och 16 pixlars text. Beskriv 32 som **mål för en OCR-box i sidrenderingen**. Behåll dagens parametrar i första porten. Justeringar av sidupplösning, rutstorlek och överlapp kan redan göras med kommandoradsflaggorna. | Återskapa nedanstående kalibrering och välj nytt standardvärde bara om ritningsprovet motiverar det. |

Fristående installation använder skillens befintliga `name`, `description`, `scripts/` och `references/`. Ett OpenAI-pluginmanifest behövs först om pluginpaketering väljs. Anthropic-manifestet kan ligga kvar för Claude. Python-modulerna importerar inget Anthropic-SDK. Den gemensamma gränsen mot modellen är `PageExtraction`. För samma extraktioner ska grafens innehåll, frågor, QA och export därför kunna verifieras med befintlig kod. `pyproject.toml`:s projektbeskrivning och bilden i README kan uppdateras redaktionellt om båda plattformarna ska annonseras. [Datamodell](../skills/pid-to-kg/scripts/pidkg/models.py) · [Befintligt pluginmanifest](../.claude-plugin/plugin.json) · [OpenAI om skills](https://learn.chatgpt.com/docs/build-skills)

### ChatGPT i webbläsaren

1. **Distribution.** Paketera samma skill med skript och referenser. Prova skilluppladdning i den avsedda arbetsytan där funktionen är aktiverad. För distribution som plugin används ett eget OpenAI-manifest i `.codex-plugin/plugin.json` med skillen i `skills/`. Ta med MIT-licensen vid distribution. Dokumentera den valda installationen i README först efter ett prov från en ny chatt. [Skills i ChatGPT](https://help.openai.com/en/articles/20001066-skills-in-chatgpt) · [Pluginpaketering](https://learn.chatgpt.com/docs/build-plugins) · [Licens](../LICENSE)
2. **Körmiljö.** Lägg till en webbvariant av stegen i `SKILL.md`. Kontrollera att körmiljön kan öppna de uppladdade PDF-byten, importera Pydantic 2, Pillow, PDFium, DuckDB och NetworkX och faktiskt utföra en OCR-avläsning med modellfilerna. ChatGPT:s dokumenterade Python-miljö för dataanalys saknar utgående webbanrop. Planera därför inte installation från PyPI under en sådan körning. Om ett bibliotek saknas, pröva ett kompatibelt förberett paket där produktens distributionsform tillåter det. En separat tjänst för Python-stegen är ett större arkitekturbeslut om de inte kan köras på plats. [OpenAI om dataanalysmiljön](https://help.openai.com/en/articles/8437071-data-analysis-with-chatgpt)
3. **Bild- och filöverföring.** Ersätt lokala sökvägar i användarflödet med sessionens indata, skrivyta och nedladdningsbara filer. Bevisa att skriptets PNG faktiskt når modellens synfunktion, även för en andra zoom efter QA. En sökväg eller ett OCR-svar räcker inte för linjespårning. Lämna ut `graph.json`, `graph.duckdb`, `summary.md` och `qa_report.md`, exempelvis i en ZIP. För återupptagning behövs också käll-PDF och motsvarande manifest, bildmetadata och bladextraktioner, med sökvägen till PDF:en återställd i den nya sessionen. Ta med bilder och ledtrådar från textlager/OCR, eller återskapa dem med samma sidurval, rotation, renderingsmått, rutstorlek och överlapp innan QA fortsätter. Kontrollera att en nedladdad DuckDB-fil går att öppna och fråga. [Manifest och PDF-sökväg](../skills/pid-to-kg/scripts/probe_pdf.py) · [Filhantering i ChatGPT](https://help.openai.com/en/articles/8555545-uploading-files-and-audio-to-chatgpt)

## Bildskalning från PDF till modell

**Sidans upplösning och modellens bildstorlek är två olika reglage.** En vektor-PDF renderas på nytt vid önskad upplösning. För ett skannat blad börjar PDFium vid skattad ursprunglig upplösning. Med inbyggd rasterlångsida `N` och begärd långsida `L` blir skalfaktorn `min(2,5; max(1; L/N))` före kontrastjustering. Ett högre värde på `--long-edge` kan ge mer verklig detalj ur en vektorritning. På en skanning interpoleras de ursprungliga bildpunkterna. Hybridblad går via vektorrenderingen och inbäddade rasterdelar behåller sin begränsade källdetalj. [PDF-rasterisering och rutnät](../skills/pid-to-kg/scripts/pidkg/pdf.py)

Med fungerande OCR gör skriptet en förhandsrendering med **begärd** långsida 2 400 px och skattar den nedre kvartilen av detekterade textboxhöjder. Om kvartilen är `h` px och förhandsbildens faktiska långsida är `P`, väljs `round(32 × P / h)`, begränsat till 3 000–8 000 px. Vid uteblivet textmått är målet 5 000 px för vektorvägen och 4 000 för rastervägen. En övergripande titel, en sällsynt liten tagg och en OCR-box kan ha helt olika höjd. [Upplösningsval](../skills/pid-to-kg/scripts/render_pages.py) · [OCR-mätningen](../skills/pid-to-kg/scripts/pidkg/ocr.py)

Översikten skalas till 1 568 px långsida. Standardrutorna tas direkt ur sidbilden vid 1 400 px med minst cirka 200 px överlapp när det finns angränsande rutor. Per axel av längd `D > T` används `ceil((D − O)/(T − O))` rutor för rutstorlek `T` och överlapp `O`. Första och sista rutan läggs vid kanterna så verkligt överlapp kan bli större. `crop.py` förstorar ett område i den sparade `page.png` till normalt 1 400 px långsida. För mer källdetalj ur CAD krävs ny PDF-rendering. Rutnamn och kalibrerade koordinater måste följa den renderingsomgång som användes vid extraktionen. [Rutnät](../skills/pid-to-kg/scripts/pidkg/pdf.py) · [Beskärning](../skills/pid-to-kg/scripts/crop.py)

| Bildväg | Vad som är belagt | Konsekvens för porten |
|---|---|---|
| Claude | Repots kommentar antar att en kvadrat på 1 400 px visas i cirka 1 090 px. Anthropics bildbudget varierar mellan modeller och detaljnivåer. Vid exakt den antagna skalningen blir en 32 px box **24,9 px**. Kommentaren nämner 16 px men koden anger ingen omräkning från box till teckenhöjd. | Använd 1 090 px som namngivet jämförelsescenario. Mät de synliga tecknen och kontrollera den faktiska Claude-versionen vid jämförelse. [Renderer](../skills/pid-to-kg/scripts/render_pages.py) · [Anthropics bildguide](https://platform.claude.com/docs/en/build-with-claude/vision) |
| Lokal Codex | Den granskade offentliga klientkoden `0.169.0-alpha.11` har en budget för originalbilder på 6 000 px långsida och 10 000 block om 32 × 32 px. En annan kodväg använder 2 048 px och 2 500 block. Båda rymmer en ruta på 1 400 × 1 400, som kräver 44 × 44 = 1 936 block. Lokalt CLI rapporterade `0.169.0-alpha.11.1`. Den hanterade sessionens bildväg har inte kunnat identifieras från versionsnumret. | Låt modellen öppna originalrutan först. I den här hanterade sessionens verktyg `functions.exec` → `view_image` innehöll returnerad PNG-data **1 400 × 1 400** för rutan, **1 568 × 1 108** för översikten och **5 000 × 3 532** för sidbilden. Vi läste måtten ur PNG-huvudet vid verktygets utgång. Modellens slutliga interna bildstorlek är okänd. [Klientens budget](https://github.com/openai/codex/blob/rust-v0.169.0-alpha.11/codex-rs/utils/image/src/lib.rs#L64-L75) · [Val av kodväg](https://github.com/openai/codex/blob/rust-v0.169.0-alpha.11/codex-rs/core/src/image_preparation.rs#L388-L415) |
| ChatGPT webb | OpenAI anger att bilder kan skalas om före analys och anger en uppladdningsgräns på 20 MB per bild. Någon garanterad analysupplösning för de ritningsbilder som webbskillen visar framgår inte. | Kör samma blinda avläsning via **den installerade webbskillens** faktiska bildväg. Registrera klient, vald modell, datum, filens mått och observerat läsresultat. [ChatGPT Image Inputs FAQ](https://help.openai.com/en/articles/8400551-chatgpt-image-inputs-faq) |
| Direkt OpenAI-API, om det väljs senare | API:t har modellspecifika regler för `detail` och pixel-/blockbudget. | En API-port kan välja detaljnivå för en stödd modell. Det behövs ingen API-integration för den föreslagna lokala porten. [Modellspecifik bildbehandling](https://developers.openai.com/api/docs/guides/images-vision#model-sizing-behavior) |

## Kalibreringsprov med syntetisk ritning

Vi använde ett A1-liknande blad (2 384 × 1 684 PDF-punkter) med rörsymboler, förgreningar och tolv slumpade taggar. Två taggar vardera sattes i Helvetica 2,5, 3,5, 5, 7, 10 och 14 pt. Samma blad provades som vektor-PDF och som gråskaleskanning, rasteriserad vid 100 dpi och JPEG-komprimerad med kvalitet 77. Skanningen hade 3 312 × 2 339 ursprungliga bildpunkter. Varje körning använde repots verkliga `probe_pdf.py` och `render_pages.py` med rotation låst till 0°. Det isolerar bildstorleken från orienteringsvalet. Adaptivt betyder här att OCR styr sidstorleken. Alla bilder och taggar i provet är syntetiska. Mätmiljön var Windows med Python 3.12.16, pypdfium2 5.15.2 (PDFium 159.0.8149.0), Pillow 13.0.0, RapidOCR-ONNXRuntime 1.4.4, ONNX Runtime 1.32.0, OpenCV 5.1.0.95 och NumPy 2.5.5. Adaptiv upplösning beror på vad OCR hittar. Installerad NumPy påverkar även hur `ink_fraction` filtrerar kantlinjer vid tomrutekontrollen.

| Källa | Begärd långsida | Rutstorlek / överlapp (px) | Mätt sidbild (px) | Kolumner × rader | Icke-tomma rutor att granska |
|---|---:|---:|---:|---:|---:|
| Vektor | 5 000 | 1 400 / 200 | 5 000 × 3 532 | 4 × 3 | 12 |
| Vektor | 5 000 | 960 / 160 | 5 000 × 3 532 | 7 × 5 | 35 |
| Vektor | Adaptiv | 1 400 / 200 | 6 378 × 4 506 | 6 × 4 | 24 |
| Vektor | 8 000 | 1 400 / 200 | 8 000 × 5 652 | 7 × 5 | 30 |
| Skanning | 5 000 | 1 400 / 200 | 5 000 × 3 531 | 4 × 3 | 12 |
| Skanning | 5 000 | 960 / 160 | 5 000 × 3 531 | 7 × 5 | 35 |
| Skanning | Adaptiv | 1 400 / 200 | 4 908 × 3 466 | 4 × 3 | 12 |
| Skanning | 8 000 | 1 400 / 200 | 8 000 × 5 650 | 7 × 5 | 30 |

Vid 5 000/1 400 blev det verkliga överlappet på vektorbladet 200 px i sidled och 334 px i höjdled. Vid 5 000/960 blev det 286–287 respektive 317 px. I försöket med mindre rutor ökade alltså antalet bildöppningar med en faktor 2,92. Antalet är ett mått på granskningsarbete. Det mäter varken API-kostnad eller svarstid.

### Vad gick att läsa?

För varje förstavisning fick läsaren samma uppgift att transkribera tolv taggar uppifrån och ned utan OCR, extraherat PDF-textlager eller facit. En osäker bokstav eller siffra räknades som fel på hela taggen. Tabellen visar lokal modellavläsning via `view_image`. Raderna med 1 090 px är en **avsiktligt skapad Lanczos-nedskalning**. De representerar ett antagande i repot, inte en uppmätt intern ChatGPT- eller Claude-gräns.

| Indata till läsaren | Exakt rätt | Vad provet visar |
|---|---:|---|
| Vektor 5 000, 1 400-ruta nedskalad till 1 090, två första läsningar | 10/12 respektive 10/12 | Båda läste 3,5–14 pt rätt. Båda markerade osäkerhet i de två taggarna på 2,5 pt. |
| Samma läsare, därefter utsnitt av den sparade sidbilden, långsida 1 090 | 10/12 respektive 12/12 | Ett riktat utsnitt hjälpte en läsare till full träff. Denna andra läsning hade redan sett samma taggar. |
| Vektor 5 000, ursprunglig 1 400-ruta, ny läsare | 11/12 | En osäker siffra på översta raden. |
| Vektor 8 000, ursprunglig 1 400-ruta, ny läsare | 12/12 | Läsaren angav båda 2,5-pt-taggarna rätt i denna körning. |
| Skanning 100 dpi, sidrendering 5 000, ursprunglig 1 400-ruta, ny läsare | 7/12 | Fem taggar blev helt eller delvis osäkra trots samma rutstorlek som för vektorn. |

De tre sista läsningarna använde var sin ny GPT-6.2 Sol-session med medelhög resonemangsnivå. De två läsningarna av 1 090-bilden gjordes i en Astra- och en Sol-session. Varje läsare öppnade rutan innan facit visades. Proven använder samma text på alla bildvarianter, ett teckensnitt och ett blad. Uppmätt taggläsning räcker inte för att fastställa en allmän teckenstorleksgräns, korrekt linjespårning eller likvärdighet med Claude eller webben.

För 2,5-pt-taggarna i vektorfallet vid 5 000 var den synliga bläckhöjden cirka **3 px** i den nedskalade rutan, **4 px** i den ursprungliga 960-rutan och **8 px** i det förstorade detaljutsnittet. Måttet är höjden på mörka pixlar vid gråvärde <160. Översikten vid 1 090 px hade inga pixlar under den tröskeln för dessa taggar. Läs därför orienteringen från översikten och de minsta taggarna från detaljerna.

**Möjlig ändring efter pilot.** För vektorritningar kan `crop.py` få en valbar rendering direkt ur PDF:en för den begärda rutan och upplösningen. I ett separat PDFium-prov gav ett direkt utsnitt 845 × 1 090 px. På den minsta taggen innehöll det 146 pixlar mörkare än gråvärde 128, jämfört med 28 i samma område efter förstoring av den sparade sidbilden. Detta mäter kontrast och sampling. Något separat blint träffmått för direktutsnittet togs inte. Om funktionen införs, testa samtliga fyra rotationer, beskärning vid bladkant och återföring till bladets normaliserade koordinater. Återanvänd rastervägen för skanningar. [Befintligt beskärningsverktyg](../skills/pid-to-kg/scripts/crop.py)

## Prov som avgör funktionell likvärdighet

| Prov | Genomförande och godkännande |
|---|---|
| Deterministisk referens | På basversionen passerade **78 tester** i en separat Python 3.12-miljö. Kör sviten igen på porten. Bygg samma tvåbladsfixture på båda värdarna och jämför noder, riktade kanter, egenskaper, bladreferenser och QA-fynd efter sortering. Fråga DuckDB om en känd flödesväg och reglerslinga. Prova GraphML och Neo4j-export. Jämför resultaten semantiskt, eftersom databaser kan ha olika binär representation. [Tester och fixture](../tests/) · [Frågeverktyg](../skills/pid-to-kg/scripts/kg_query.py) · [Export](../skills/pid-to-kg/scripts/export.py) |
| Visuell jämförelse | Välj ritningar som får användas på de tänkta plattformarna. Inkludera en tät CAD-export med textlager, text ritad som konturer, en skanning samt en förbindelse mellan två blad. Ta med flödespil, korsning utan förbindelse, ventil före/efter mätuttag, styrsignal och liknande taggar som skiljer en siffra. Låt en ingenjör sätta facit före körning. Kör Claude och varje vald målmiljö på samma revisioner med oberoende sessioner. Upprepa med nya sessioner för att synliggöra variation. |
| Mätning och beslut | Kör `eval_kg.py` mot facit för taggarnas och utrustningsflödenas precision/recall, instrumentplacering och slingmedlemmar. Granska även exakt rörsegment, ordning och riktning samt alla länkar över bladgränser. Skriptets instrumentmått arbetar mot närmaste utrustning och kan godkänna ett uttag på fel sida om en ventil. Föreslaget pilotkriterium är noll felaktigt angivna kritiska samband. Varje kritiskt samband som ingenjören bedömt som läsbart ska återges rätt. Ett oläsligt kritiskt samband ska redovisas och hindrar automatisk acceptans av det berörda bladet. Sätt toleranserna för övriga mått före jämförelsen och redovisa felaktiga respektive oläsliga poster separat. [Poängsättning](../skills/pid-to-kg/scripts/eval_kg.py) |
| Slutleverans | Kontrollera att varje valt blad täcks, att oläsliga taggar är markerade, att QA-fel är åtgärdade och att återstående varningar är förklarade. I webbfallet ska en ny chatt klara filinläsning, bildåterkoppling, ett omtag efter QA, nedladdning och frågor mot grafen. Redovisa resultat för varje miljö separat. [Repots granskningsprocedur](../skills/pid-to-kg/references/extraction_procedure.md) |

## Föreslaget första utförande

Börja med lokal Codex och **behåll adaptiv sidrendering samt 1 400/200 px**. Använd översikten för orientering och gå direkt till ett utsnitt när en siffra eller korsning är osäker. Prova högre vektorrendering på det berörda bladet om utsnittet fortfarande är suddigt. Ny rendering från vektorkällan gav bättre läsbarhet i provet, medan mindre standardrutor nästan tredubblade arbetet på samma sidbild. För skannade ritningar prioriteras källans detaljrikedom. Välj webbläsarport om målgruppens arbetsyta klarar körproven och ger samma tillgång till utsnitt och filer. Kvarstående beslut gäller målmiljö, åtkomst till representativa ritningar och vilka avvikelser som är acceptabla för användningen.

## Bilaga – upprepa bildprovet

Spara Python-blocket nedan som `fixture.py` i en tom arbetskatalog. Det skriver kalibreringsparet `vector.pdf` och `scan100.pdf` med de uppmätta taggarna. Använd Python 3.12 med repots beroenden. Följande PowerShell-exempel använder en befintlig Pixi-miljö. Ange absoluta sökvägar i `$miljo` (manifestet) och `$skript` (skillens `scripts`-katalog).

```powershell
pixi run --manifest-path $miljo python fixture.py
foreach ($pdf in 'vector', 'scan100') {
    $fall = @{
        'adaptiv' = @()
        '5000'    = @('--long-edge', '5000')
        '960'     = @('--long-edge', '5000', '--tile', '960', '--overlap', '160')
        '8000'    = @('--long-edge', '8000')
    }
    foreach ($namn in $fall.Keys) {
        $arbete = "$pdf-$namn"
        $parametrar = $fall[$namn]
        pixi run --manifest-path $miljo python "$skript/probe_pdf.py" "$pdf.pdf" --work $arbete
        pixi run --manifest-path $miljo python "$skript/render_pages.py" $arbete --rotation 0 @parametrar
    }
}
pixi run --manifest-path $miljo python "$skript/crop.py" vector-5000 --page 1 --bbox 0.018876 0.038599 0.098574 0.184086 --size 1400 --out crop.png
```

Läs `pages/page_01/tiles.json` för rutantal, mått, faktiska överlapp och `blank`. Använd `tiles/r1c1.png` för taggarna. För 1 090-provet skalas PNG:n proportionellt med Pillow/Lanczos till högst 1 090 px långsida. Beskärningens exakta normaliserade ruta är `(45/2384, 65/1684, 235/2384, 310/1684)`. Vid kontrollen gav bilagans båda PDF-filer samma SHA-256 som försöksfilerna. För `vector.pdf` var den `8679a4017221b99a0436737b6b5087080defc3a5dda611951f994ab2f94fe041` och för `scan100.pdf` var den `04f63059af5c15afefd07e034b905818a32be14fff56abc67254fecb52820976`. För en ny blind avläsning bör en annan person generera nya likvärdiga taggar och behålla facit tills läsarens svar är sparade.

<details>
<summary>Python-kod som återskapar provets två PDF-filer</summary>

```python
from io import BytesIO
from pathlib import Path

import pypdfium2 as pdfium

root = Path.cwd()
width, height = 2384, 1684
font_sizes = (2.5, 3.5, 5, 7, 10, 14)
tags = (
    "FCV-9733A", "FT-5487S", "XV-3307D", "TT-8884B", "P-1818A", "TT-5584S",
    "PT-0449S", "TT-7045B", "V-6626A", "V-4156S", "V-0007A", "TT-4319B",
)
vector_parts = ["0 G 0 g .6 w"]
def line(x1, y1, x2, y2):
    vector_parts.append(f"{x1} {height-y1} m {x2} {height-y2} l S")
def rect(x, y, w, h):
    vector_parts.append(f"{x} {height-y-h} {w} {h} re S")
def txt(x, y, size, label):
    vector_parts.append(f"BT /F1 {size} Tf {x} {height-y} Td ({label}) Tj ET")
def pdf(objects, dest):
    data = bytearray(b"%PDF-1.4\n%\xE2\xE3\xCF\xD3\n")
    offsets = [0]
    for index, content in enumerate(objects, 1):
        offsets.append(len(data))
        data.extend(f"{index} 0 obj\n".encode())
        data.extend(content)
        data.extend(b"\nendobj\n")
    start = len(data)
    data.extend(f"xref\n0 {len(offsets)}\n0000000000 65535 f \n".encode())
    for offset in offsets[1:]:
        data.extend(f"{offset:010} 00000 n \n".encode())
    data.extend(f"trailer\n<< /Size {len(offsets)} /Root 1 0 R >>\nstartxref\n{start}\n%%EOF\n".encode())
    dest.write_bytes(data)
def stream(data, attrs=""):
    return f"<< /Length {len(data)} {attrs} >>\nstream\n".encode() + data + b"\nendstream"
catalog = b"<< /Type /Catalog /Pages 2 0 R >>"
pages = b"<< /Type /Pages /Count 1 /Kids [3 0 R] >>"
base_page = f"/Type /Page /Parent 2 0 R /MediaBox [0 0 {width} {height}] /Contents 5 0 R"
rect(12, 12, width-24, height-24)
txt(55, 40, 16, "SYNTHETIC DRAWING - PIPE AND INSTRUMENTATION")
for index in range(12):
    y = 81 + 19 * index
    size = font_sizes[index//2]
    tag = tags[index]
    line(63, y+4, 137, y+4)
    line(63, y+4, 63, y+21 if index < 11 else y+7)
    if index % 2 == 0:
        line(91, y, 100, y+4)
        line(91, y+8, 100, y+4)
        line(100, y+4, 109, y)
        line(100, y+4, 109, y+8)
        line(91, y, 91, y+8)
        line(109, y, 109, y+8)
    else:
        rect(94, y+1, 8, 6)
    txt(145, y+1, size, tag)
for yy in (475, 695, 915, 1135, 1355):
    for xx in (100, 460, 820, 1180, 1540, 1900):
        rect(xx, yy, 55, 85)
        line(xx+55, yy+42, xx+255, yy+42)
        line(xx+155, yy+42, xx+155, yy-32)
        line(xx+250, yy+39, xx+255, yy+42)
        line(xx+250, yy+45, xx+255, yy+42)
        rect(xx+143, yy-55, 24, 24)
        txt(xx+3, yy+41, 10, "VESSEL")
        txt(xx+148, yy-40, 9, "PI")
        txt(xx+175, yy+35, 8, "PROCESS LINE")
rect(1930,1510,420,132)
txt(1955,1550,15,"SYNTHETIC VALIDATION SHEET")
txt(1955,1595,12,"SIZE A1   REV 0")
vector = root / "vector.pdf"
pdf([catalog, pages, f"<< {base_page} /Resources << /Font << /F1 4 0 R >> >> >>".encode(),
    b"<< /Type /Font /Subtype /Type1 /BaseFont /Helvetica >>",
    stream("\n".join(vector_parts).encode())], vector)
doc = pdfium.PdfDocument(vector)
bitmap = doc[0].render(scale=100/72, grayscale=True).to_pil().copy()
buf = BytesIO()
bitmap.save(buf, format="JPEG", quality=77)
scan = root / "scan100.pdf"
pdf([catalog, pages, f"<< {base_page} /Resources << /XObject << /Im0 4 0 R >> >> >>".encode(),
    stream(buf.getvalue(), f"/Type /XObject /Subtype /Image /Width {bitmap.width} /Height {bitmap.height} /ColorSpace /DeviceGray /BitsPerComponent 8 /Filter /DCTDecode"),
    stream(f"q {width} 0 0 {height} 0 0 cm /Im0 Do Q".encode())], scan)
```

</details>

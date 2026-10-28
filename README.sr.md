# Predviđanje ozbiljnosti saobraćajnih nezgoda metodama mašinskog učenja

**Jezik:** [English](README.md) | Srpski

- **Završni rad:** *Primjena metoda mašinskog učenja za predviđanje ozbiljnosti saobraćajnih nezgoda*
- **Autor:** Matea Masleša, Visoka škola strukovnih studija za informacione tehnologije (ITS), Beograd, 2026.
- **Mentor:** dr Đorđe Petrović

## Cilj

Na osnovu okolnosti saobraćajne nezgode (mjesto, vrijeme, tip i opis nezgode) predviđa se njena **posljedica** u tri klase: materijalna šteta, povrijeđeni i poginuli. Rad poredi četiri modela i posebno se bavi izrazitom neuravnoteženošću klasa.

## Podaci

- **Izvor:** MUP Republike Srbije, *Podaci o saobraćajnim nezgodama po policijskim upravama i opštinama za 2025. godinu*, Portal otvorenih podataka ([link](https://data.gov.rs/sr/datasets/podatsi-o-saobratshajnim-nezgodama-po-politsijskim-upravama-i-opshtinama/)).
- **Veličina:** 33.415 nezgoda iz 2025. godine, 9 originalnih kolona.
- **Podaci nisu u repozitoriju.** Uputstvo je u `data/README.md`.
- **Raspodjela klasa:** materijalna šteta oko 59%, povrijeđeni oko 39%, **poginuli samo 1,4% (463 nezgode)**.

## Metodologija

1. **Eksplorativna analiza:** raspodjela po policijskim upravama i opštinama, tipovima nezgoda, mjesecima, satima i danima u sedmici (grafici u `results/`).
2. **Atributi (11):** kategorijski: policijska uprava, opština, tip nezgode, opis nezgode, dan u sedmici. Numerički: geografska dužina i širina, te mjesec i sat kodirani ciklično (sin/cos).
3. **Podjela podataka:** 80/20, stratifikovano (`random_state=42`): 26.732 za trening, 6.683 za test (92 nezgode sa poginulima). Podjela se radi **prije** enkodiranja i balansiranja, da test skup ne bi bio "zagađen".
4. **Neuravnoteženost klasa:** SMOTENC (samo na trening skupu; poginuli sa 371 na 15.855 primjera) uz težine klasa u modelima.
5. **Modeli:** Logistička regresija, Decision Tree, Random Forest i CatBoost.
6. **Evaluacija:** accuracy, weighted i macro F1, te preciznost, odziv i F1 za klasu "poginuli". Dodatno 5-struka stratifikovana unakrsna validacija za Random Forest (SMOTENC unutar pipeline-a).
7. **Provjera curenja informacija:** provjereno je da opis nezgode ne sadrži izravno riječ "pogin" za klasu poginulih (0,00%).

## Rezultati (test skup, 6.683 nezgode)

| Model | Accuracy | Weighted F1 | Macro F1 | Preciznost (poginuli) | Odziv (poginuli) |
|---|---|---|---|---|---|
| Logistička regresija | 0,64 | 0,69 | 0,48 | 0,04 | **0,42** |
| Decision Tree | 0,67 | 0,69 | 0,48 | 0,03 | 0,20 |
| Random Forest | 0,73 | 0,73 | 0,51 | 0,06 | 0,11 |
| **CatBoost** | **0,74** | **0,74** | **0,53** | 0,10 | 0,13 |

Tabela je u `results/rezultati_modela.csv`, a matrice konfuzije i poređenje modela u `results/` (`14`–`20`).

**Glavni nalazi**
- **CatBoost** ima najbolju ukupnu tačnost (0,74) i najbolji macro F1 (0,53).
- **Logistička regresija** pronalazi najviše nezgoda sa poginulima (odziv 0,42), ali uz veoma malu preciznost (0,04), tj. većinu "upozorenja" čine lažni alarmi.
- Unakrsna validacija Random Forest-a: weighted F1 0,734 ± 0,003, odziv za poginule 0,076 ± 0,011. Rezultati su stabilni, ali odziv za poginule ostaje nizak.
- Izbor modela zavisi od cilja: za ukupnu tačnost CatBoost, a za hvatanje rijetkih teških nezgoda Logistička regresija, uz cijenu lažnih alarma.

## Ograničenja

- **Klasa "poginuli" se i dalje loše predviđa.** Preciznost je 0,03–0,10 i odziv 0,11–0,42, pa model nije pogodan za samostalno upozoravanje.
- Atributi opisuju samu nezgodu (uključujući njen opis), pa model **procjenjuje ozbiljnost iz okolnosti nezgode**, a ne predviđa je unaprijed.
- Random Forest je prilagođen trening podacima (tačnost na treningu 0,92, na testu 0,73).
- Samo jedna godina podataka i jedna nasumična stratifikovana podjela. Test skup ima samo 92 nezgode sa poginulima, pa su procjene za tu klasu nesigurne.
- Hiperparametri nisu sistematski optimizovani.

## Struktura i pokretanje

```
data/        uputstvo za preuzimanje podataka
notebooks/   Diplomski_kod.ipynb (kod sa sačuvanim rezultatima)
results/     grafici i tabela rezultata
```

```bash
pip install -r requirements.txt
jupyter notebook notebooks/Diplomski_kod.ipynb
```

Prije pokretanja preuzmi podatke u folder `data/` (vidi `data/README.md`).

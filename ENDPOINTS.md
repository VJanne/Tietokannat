# My Festival API — endpoint-dokumentti / endpoint document

> FI: Täytä tämä pohja ja palauta se Moodleen (Markdown tai PDF, 1–2 sivua). Korvaa kaikki `<…>`-kohdat.
> Esimerkkivastaukset kopioit oman julkisen rajapintasi oikeista vastauksista (selain tai `curl.exe`/`curl`).
>
> EN: Fill in this template and submit it to Moodle (Markdown or PDF, 1–2 pages). Replace every `<…>`.
> Copy the example responses from your own public API's real responses (browser or `curl.exe`/`curl`).

**Tekijä / Author:** <nimi / name> Janne Väljä
**Julkinen osoite / Public URL:** `https://festival-api-festival-valjajan.2.rahtiapp.fi/`
**GitHub:** `https://github.com/VJanne/Tietokannat`
**Tarkistimen tulos / Checker result:** `20/20 checks passed` (liitä koko tuloste palautukseen / attach the full output)

---

## 1. Endpointit / Endpoints

| Metodi / Method | Polku / Path | Onnistuu / Success | Virheet / Errors |
|---|---|---|---|
| GET | `/api/artists` | 200 | – |
| GET | `/api/days` | 200 | – |
| GET | `/api/stages` | 200 | – |
| GET | `/api/artists/{id}/events` | 200 | 404 |
| GET | `/api/timetable?day=YYYY-MM-DD` | 200 | 400 |
| POST | `/api/events` | 201 | 400 |
| DELETE | `/api/events/{id}` | 204 | 404 |

## 2. Esimerkit / Examples



### GET /api/artists

```bash
curl https://festival-api-festival-valjajan.2.rahtiapp.fi/api/artists
```
```json
[{"artist_id":10,"country":"UK","name":"Adele"},{"artist_id":17,"country":"FIN","name":"Alma"}]...
```

### GET /api/artists/{id}/events
```bash
curl https://festival-api-festival-valjajan.2.rahtiapp.fi/api/artists/10/events
```
```json
[{"day":"2025-07-05","event_id":1,"stage":"Main A","starts":"14:15:00"},{"day":"2025-07-05","event_id":27,"stage":"Club","starts":"17:15:00"}]....
```


Tuntematon id / unknown id → `<tilakoodi ja runko / status code and body>`
```bash
curl https://festival-api-festival-valjajan.2.rahtiapp.fi/api/artists/10000/events
```
```json
{"error":"artist 10000 not found"}
```




### GET /api/timetable?day=YYYY-MM-DD
```bash
curl https://festival-api-festival-valjajan.2.rahtiapp.fi/api/timetable?day=2025-07-06
```
```json
[{"artist":"Imagine Dragons","date":"2025-07-06","ends":"14:40:00","stage":"Club","starts":"14:15:00"},{"artist":"Adele","date":"2025-07-06","ends":"15:00:00","stage":"Main A","starts":"14:15:00"}]...
```


Virheellinen päivämäärä / invalid date → `<tilakoodi ja runko / status code and body>`
```bash
curl https://festival-api-festival-valjajan.2.rahtiapp.fi/api/timetable?day=2025-4323
```
```json
{"error":"invalid date, use YYYY-MM-DD"}
```



### POST /api/events
```bash
curl -i -X POST https://festival-api-festival-valjajan.2.rahtiapp.fi/api/events \
     -H "Content-Type: application/json" \
     -d '{"artist_id": 1, "stage_id": 1, "day_id": 1, "start_dt": "2026-07-10 20:00:00"}'
```
```json
<artist_id         : 1
day_id            : 1
duration_min      : 45
end_dt            : 2026-07-10 20:45:00
event_id          : 127
event_type_id     : 1
previous_event_id :
stage_id          : 1
start_dt          : 2026-07-10 20:00:00>
```
Puuttuva kenttä / missing field → `{"error":"missing fields: stage_id, day_id"}` · Olematon artist_id / unknown artist_id → `{"error":"integrity error: UNIQUE constraint failed: Event.day_id, Event.stage_id, Event.start_dt"}`

### DELETE /api/events/{id}
```bash
curl -X DELETE https://festival-api-festival-valjajan.2.rahtiapp.fi/api/events/10
```
Onnistuu / success → `<>` · Tuntematon id / unknown id → `{"error":"event 10 not found"}`



## 3. Julkaisu Rahtiin / Deploying to Rahti (3–6 lausetta / sentences)

Rahti haki koodin GitHub-reposta ja rakensi Dockerfilen perusteella konttiimagen (build). Rakennetusta imagesta käynnistyi Pod, jota Deployment ylläpitää (deployment), ja Route tarjoaa sille julkisen HTTPS-osoitteen (route). Osoitteen löysin Topology-näkymästä nuolikuvakkeesta; se päättyy `rahtiapp.fi`. Sovellusta ajaa gunicorn portissa 8080.

**Ongelma ja ratkaisu
Tarkistin antoi paikallisesti 20/20, mutta julkista osoitetta vasten osa testeistä epäonnistui. Syy oli, että käytin väärää osoitetta. Kopioin osoitteen uudelleen Topology-näkymästä, ja kaikki testit menivät läpi.

## 4. Tietoturva / Security (2–4 lausetta / sentences)

SQL-injektio estetään parametrisoiduilla kyselyillä: käyttäjän syöte (`id`, `day`, POST-kentät) annetaan tietokannalle aina `?`-paikkamerkkien arvoina, eikä sitä koskaan liitetä SQL-merkkijonoon. Lisäksi päivämäärä ja `start_dt` validoidaan ja POST-kenttien tyypit tarkistetaan ennen kantaa.

Ennen oikeaa tuotantokäyttöä rajapinta tarvitsisi ainakin: autentikoinnin ja oikeuksienhallinnan POST- ja DELETE-reiteille (nyt kuka tahansa voi muokata ohjelmaa), sekä lokituksen ja seurannan. HTTPS on jo käytössä Rahdin edge-routen ansiosta.

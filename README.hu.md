🇬🇧 [English](README.md) | 🇭🇺 **Magyar**

# ⛅ Időjárás-állomás Feltöltő – Időkép, Metnet, Weather Underground (Home Assistant Blueprint)

[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20me%20a%20coffee-FFDD00?logo=buymeacoffee&logoColor=black)](https://buymeacoffee.com/adkulik)

Ezzel a Home Assistant sablon-automatizmussal (angolul: Blueprint) egy grafikus felületen keresztül kötheted be az okosotthonod meteorológiai szenzorait egyszerre több időjárási hálózatba:

* **Időkép** (Normál és Pro fiók, Időkép vagy Weather Underground formátumban)
* **Metnet**
* **Weather Underground**

Minden oldalt külön kapcsolhatsz be, a szenzorokat pedig csak egyszer kell megadni. A sablon a háttérben elvégzi a nehezét:

* Az adatokat minden oldalnak a saját formátumában és mértékegységeiben küldi (°C/°F, hPa/inHg, m/s/mph, mm/inch, tizedesvessző vagy pont).
* A bekapcsolt oldalak párhuzamosan kapják az adatot. Ha az egyik nem elérhető, a többire attól még felmegy.
* A nem elérhető (unavailable/unknown) szenzorok kimaradnak a beküldésből, így nem kerülnek fel hamis nullák.

> A korábbi, csak Időképre feltöltő **2.x sablon** (`idokep.hu-uploader.yaml`) továbbra is ebben a repóban van és működik, de új funkciót már csak a 3.x kap. Az átállást lásd lent: [Átállás a 2.x Időkép Feltöltőről](#-átállás-a-2x-időkép-feltöltőről).

---

## ❗ Fontos figyelmeztetés: Csak hiteles adatokat tölts fel!

Ezeknek a hálózatoknak a térképeit és előrejelzéseit nagyon sokan használják nap mint nap. Kérjük, **csak és kizárólag akkor indíts adatfeltöltést, ha a szenzoraid elhelyezése megfelel a meteorológiai szabványoknak!** A rossz helyre tett műszerek (pl. napon lévő hőmérő, fal melletti szélmérő) hibás adatokat generálnak, amivel rontod a hálózat megbízhatóságát.

### 📐 Műszerelhelyezési útmutató
* **Hőmérő és páratartalom mérő:** 2 méter magasságban, árnyékos helyen (lehetőleg szabványos hőmérőházban vagy tányéros árnyékolóban) kell elhelyezni. Szigorúan tilos falhoz, betonhoz vagy olyan felülethez rögzíteni, ami hőt sugározhat magából!
* **Szélmérő:** Szabad, nyílt területen, a talajszinttől számítva ideálisan 10 méter magasan kell lennie. Ha erre nincs mód, akkor a környező akadályok (háztető, fák) fölé kell nyúlnia legalább 2-3 méterrel.
* **Csapadékmérő:** Nyílt, vízszintes területen, a talajtól nagyjából 1-1,5 méter magasságban. Épület, fa vagy tető ne takarja ki.
* **Légnyomásmérő:** Beltéren is elhelyezhető, de a Home Assistant mindig **tengerszintre átszámított (relatív) légnyomásértéket** küldjön, ne a szenzor által mért abszolút értéket.

---

## ⚠️ 1. Lépés: Előfeltételek

A sablon **Home Assistant 2024.10 vagy újabb** verziót igényel.

Fiók azokhoz az oldalakhoz, ahová fel szeretnél tölteni:

* **Időkép:** haladó szintű fiók, **[regisztráció itt](https://www.idokep.hu/regisztracio?szint=halado)**. Az így létrehozott fiók Normál típusú.
* **Metnet:** az állomás-beállítási oldalon, a *Weather Underground protokollal küldő állomás* résznél kapod meg az állomásazonosítót (Station ID) és a kulcsot.
* **Weather Underground:** a *My Devices* oldalon kapod meg a Station ID-t és a Station Key-t.

A sablon a Home Assistant beépített `rest_command` integrációján keresztül küldi az adatokat, ehhez egy apró bejegyzés kell a `configuration.yaml` fájlba:

1. Nyisd meg a Home Assistant `configuration.yaml` fájlját.
2. Keresd meg a `rest_command:` blokkot (ha még nincs, hozd létre a fájl végén).
3. Illeszd be az alábbi bejegyzést pontosan így, a behúzásokra figyelve:

```yaml
rest_command:
  weather_upload:
    url: "{{ url }}"
    method: get
    headers:
      User-Agent: "{{ user_agent | default('HA-Weather-Uploader') }}"
    timeout: 15
```

4. Mentsd el a fájlt, majd **indítsd újra a Home Assistantot**.

> **Mit csinál ez a blokk?** Létrehoz egy általános `rest_command.weather_upload` műveletet. A sablon minden oldalnak ezen keresztül küld, a címet és a `User-Agent` fejlécet oldalanként maga adja át. Ha később új oldal kerül a sablonba, ezt a blokkot nem kell módosítani.

> **Biztonsági megjegyzés:** a jelszavak és kulcsok az URL részei, ezért ha egy beküldés hibára fut, a Home Assistant a teljes URL-t kiírhatja a rendszernaplóba. Ez bekerülhet a diagnosztikai csomagokba és a biztonsági mentésekbe is, ezért minden oldalhoz használj külön, máshol nem használt jelszót. A beküldés mindhárom oldal felé HTTPS-en megy (a Metnetnél HTTP is választható).

---

## 🚀 2. Lépés: Telepítés

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/adkulik/ha-weather-uploader/refs/heads/main/weather-uploader.yaml)

*(Ha a gomb valamiért nem működne, másold ki ezt a linket: `https://raw.githubusercontent.com/adkulik/ha-weather-uploader/refs/heads/main/weather-uploader.yaml`, majd a Home Assistant **Sablon-automatizmusok** oldalán kattints a **Sablon-automatizmus importálása** gombra, illeszd be a linket, és kattints az **Előnézet**, végül a **Sablon-automatizmus importálása** gombra.)*

---

## ⚙️ 3. Lépés: Beállítás a felületen

1. Menj a **Beállítások → Automatizmusok és jelenetek → Sablon-automatizmusok** oldalra.
2. Keresd meg az *Időjárás-állomás Feltöltő (Időkép, Metnet, Wunderground)* sablont, és kattints az **Automatizmus létrehozása** gombra.
3. Töltsd ki a felületet:
   * **Időkép / Metnet / Weather Underground:** kapcsold be (a „Feltöltés a …” kapcsolóval) azokat az oldalakat, ahová fel szeretnél tölteni, és add meg a hozzájuk tartozó azonosítókat. A kikapcsolt vagy kitöltetlen oldalra nem megy adat.
     * *Időkép – Fióktípus:* Normál vagy Pro. Ha a fiókodat a haladó szintű regisztrációval hoztad létre, Normál fiókod van.
     * *Időkép – Feltöltési formátum (csak Normál fióknál):* Időkép formátum (a megszokott beküldés, 24 órás csapadékkal) vagy Weather Underground formátum (napi csapadékkal; a napsugárzást csak ez jeleníti meg globálsugárzásként).
     * *Metnet – Kapcsolat:* HTTPS (alapértelmezés) vagy HTTP.
   * **Ütemezés:** percenként, 2, 5, 10 vagy 15 percenként (alapértelmezés: 5 perc). Minden bekapcsolt oldalra ugyanezzel az ütemmel megy.
   * **Szenzorok:** csak a Hőmérséklet kötelező, minden más (páratartalom, légnyomás, beltéri hőmérséklet, szél, csapadék, napsugárzás, UV-index) opcionális.
4. Mentsd el az automatizmust.

### 🌬️ Formátumok és mértékegységek

A Metnet és a Weather Underground mindig, az Időkép választhatóan a Weather Underground (WU) protokollt használja. A sablon a szenzorok saját mértékegységét kiolvassa, és átváltja. A szélirány fokban (0–360) vagy égtáj-rövidítésként is megadható, angolul (N, NNE, NE …) vagy magyarul (É, ÉK, K, DK, D, DNY, NY, ÉNY).

| Adat | Időkép formátum | WU formátum (Metnet, Weather Underground, Időkép WU) |
|---|---|---|
| Hőmérséklet | `hom` (°C) | `tempf` (°F) |
| Páratartalom | `rh` (%) | `humidity` (%) |
| Harmatpont (számolt) | – | `dewptf` (°F) |
| Légnyomás | `p` (hPa) | `baromin` (inHg) |
| Beltéri hőmérséklet | `inhom` (°C) | `indoortempf` (°F) |
| Szélirány | `szelirany` (fok) | `winddir` (fok) |
| Szélsebesség | `szelero` (m/s) | `windspeedmph` (mph) |
| Széllökés | `szellokes` (m/s) | `windgustmph` (mph) |
| 1 órás csapadék | `csap1h` (mm) | `rainin` (inch) |
| 24 órás csapadék | `csap` (mm) | – |
| Napi csapadék (éjfél óta) | – | `dailyrainin` (inch) |
| Összes csapadék (számláló) | `csaptotal` (mm) | – |
| Napsugárzás | `napfeny` (egész W/m²) | `solarradiation` (W/m²) |
| UV-index | `uv` | `UV` |

* **Csapadék:** az Időkép formátum a **24 órás** (gördülő) csapadékot fogadja, a WU formátum a **napi**, éjfél óta mért csapadékot. Ha a Metnetre vagy a Weather Undergroundra is töltesz, add meg a *Napi csapadék – éjfél óta* szenzort.
* **Globálsugárzás:** a napsugárzást az Időkép csak a **WU formátumban** jeleníti meg globálsugárzásként. Ha a szenzor luxban mér, a sablon közelítőleg W/m²-re váltja.

---

## 🔁 Átállás a 2.x Időkép Feltöltőről

A 3.0 új sablon-automatizmus, ezért a régi automatizmusod addig változatlanul fut, amíg ki nem kapcsolod. Így a feltöltés az átállás alatt sem áll le:

1. Vedd fel a `configuration.yaml`-ba az 1. lépésben látható `rest_command.weather_upload` blokkot. A régi `idokep_get` bejegyzést egyelőre hagyd meg.
2. Indítsd újra a Home Assistantot.
3. Importáld a 3.0 sablont (2. lépés), és hozz létre belőle egy új automatizmust. Az Időkép részben kapcsold be a feltöltést, és add meg ugyanazokat az adatokat és szenzorokat, mint a régiben.
4. Kapcsold ki (vagy töröld) a régi, 2.x sablonból készült automatizmust.
5. Ha már semmi nem használja, töröld a `configuration.yaml`-ból a régi `idokep_get` bejegyzést, majd indítsd újra a Home Assistantot.

A régi sablon importálási címe a repó átnevezése után is működik: `https://raw.githubusercontent.com/adkulik/ha-weather-uploader/refs/heads/main/idokep.hu-uploader.yaml`

---

## 🔔 Ajánlott: értesítés az új verziókról (Blueprints Updater)

A Home Assistant magától nem szól, ha egy sablon-automatizmusnak új verziója jelenik meg. Ezt veszi át a [Blueprints Updater](https://github.com/luuquangvu/blueprints-updater) közösségi integráció: minden telepített sablonhoz frissítés-entitást hoz létre, és egy kattintással telepíthető. A sablon erre a repóra hivatkozik, ezért külön beállítás nélkül követi.

**Telepítés:** Home Assistant 2026.11-től a **Beállítások → Marketplace** oldalon (korábbi verziókban a HACS-ban) keresd meg a *Blueprints Updater*-t, telepítsd, indítsd újra a Home Assistantot, majd a **Beállítások → Eszközök és szolgáltatások → Integráció hozzáadása** gombbal add hozzá.

**Tipp:** az automatikus frissítést hagyd kikapcsolva, és frissítés előtt nézd meg a Változásnaplót. Egy-egy új verzió a `configuration.yaml` módosítását is kérheti.

---

## ☕ Támogatás

A sablon ingyenes, és az is marad. Ha hasznosnak találod, egy kávéval támogathatod a további fejlesztést:

<a href="https://buymeacoffee.com/adkulik"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me a Coffee" height="50"></a>

Rendszeres támogatásra a [GitHub Sponsors](https://github.com/sponsors/adkulik) is elérhető.

---

## 📋 Változásnapló

### 3.0 (2026-11-20) – Időjárás-állomás Feltöltő

**Új sablon** (`weather-uploader.yaml`), a repó új neve `ha-weather-uploader`.

* Egyszerre több oldalra tölt fel: **Időkép**, **Metnet** és **Weather Underground**, oldalanként külön bekapcsolható.
* A bekapcsolt oldalak párhuzamosan kapják az adatot; egy oldal hibája nem állítja meg a többit.
* Új, általános `rest_command.weather_upload` parancs (a `User-Agent` fejlécet oldalanként a sablon adja át). Új oldalhoz nem kell a `configuration.yaml`-hoz nyúlni.
* A Weather Underground formátumú mérési részt a sablon egyszer állítja össze, és ezt kapja a Metnet, a Weather Underground és az Időkép WU formátuma.
* Metnet: HTTPS kapcsolat (alapértelmezés), HTTP is választható.
* WU formátumban a harmatpontot (`dewptf`) a sablon a hőmérsékletből és a páratartalomból számolja.

### 2.8 (2026-11-20)

* A 2.x sablon leírása a 3.0-ra mutat. A beküldés változatlan.

### 2.7 (2026-11-16)

* Új **Feltöltési formátum** beállítás Normál fiókhoz: Időkép formátum vagy Weather Underground formátum (`automata.idokep.hu`, WU mezőnevek és mértékegységek).
* A napsugárzást az Időkép csak a WU formátumban jeleníti meg globálsugárzásként.
* Új, opcionális **Napi csapadék – éjfél óta** mező a WU formátumhoz.

### 2.6 (2026-11-16)

* A napsugárzás (`napfeny`) egész számra kerekítve megy fel, mert az Időkép ennél a mezőnél a tizedes értéket eldobja.

### 2.5 (2026-11-08)

* Csapadékfeltöltés, UV- és fényadatok optimalizálása.

### 2.3 (2026-11-07)

* A beküldés titkosított HTTPS-kapcsolaton megy (`https://idokep.hu`, illetve `https://pro.idokep.hu`).

### 2.2 (2026-11-07)

* Új, opcionális **Napsugárzás és UV** szekció. Lux mértékegységű szenzor értékét a sablon közelítőleg W/m²-re váltja.

### 2.1 (2026-11-01)

* Új **Fióktípus** beállítás (Normál vagy Pro). Normál fióknál az `idokep.hu`, Pro fióknál a `pro.idokep.hu` címre megy az adat.
* A beküldés típusjelölése (`tipus`) `egyeb` helyett `HA_Uploader`.

### 2.0 (2026-10-31)

**Törő változás:** a beküldés `shell_command` és curl helyett a beépített `rest_command` integrációval történik; a `configuration.yaml`-ba új bejegyzés kell.

* Ismert korlát: az Időkép válasza nem UTF-8 kódolású, ezért a sablon a válasz szövegét nem ellenőrzi.

### 1.0 (2026-10-31)

Első számozott kiadás: grafikus beállítófelület, választható feltöltési gyakoriság, szél-mértékegység átváltás, égtáj-rövidítések, a nem elérhető szenzorok kihagyása.

A curl-os 1.0 továbbra is importálható: `https://raw.githubusercontent.com/adkulik/ha-weather-uploader/refs/tags/v1.0/idokep.hu-uploader.yaml`

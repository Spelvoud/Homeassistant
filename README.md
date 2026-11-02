# Homeassistant

## Blueprints

### Shelly i4 – Hue lamp dimmen

[`blueprints/automation/spelvoud/shelly_i4_hue_dimmer.yaml`](blueprints/automation/spelvoud/shelly_i4_hue_dimmer.yaml)

Eén drukknop op een Shelly i4 bedient een Philips Hue lamp (of elke andere dimbare `light`):

| Actie            | Resultaat                                                        |
|------------------|------------------------------------------------------------------|
| Kort drukken     | Aan/uit                                                          |
| Dubbel drukken   | 100% helderheid                                                  |
| Ingedrukt houden | Dimmen tot je loslaat; richting wisselt per keer vasthouden      |

**Voorbereiding**

1. Voeg de Shelly i4 toe via de **Shelly**-integratie (Instellingen → Apparaten & diensten).
2. Zet in de Shelly web-UI (of app) de ingang op **Button** (drukknop), niet op Switch.
   Dimmen door vast te houden werkt alleen met een drukknop.
3. Optioneel: maak een helper *Schakelaar* (`input_boolean`), bijv. `input_boolean.woonkamer_dim_richting`.
   Daarmee wisselt de dimrichting elke keer dat je vasthoudt. Zonder helper geldt:
   onder 50% wordt feller, daarboven donkerder.

**Importeren**

Instellingen → Automatiseringen & scènes → Blueprints → *Blueprint importeren* en plak de URL:

```
https://github.com/spelvoud/homeassistant/blob/main/blueprints/automation/spelvoud/shelly_i4_hue_dimmer.yaml
```

(Of kopieer het bestand naar `config/blueprints/automation/spelvoud/` en herlaad de blueprints.)
Maak daarna per knop een automatisering aan op basis van de blueprint, kies de
event-entiteit van de ingang (bijv. `event.shelly_i4_input_0`) en de Hue lamp.

**Werking**

De blueprint luistert naar de event-entiteit van de Shelly (`btn_down`, `btn_up`,
`single_push`, `double_push`). Bij `btn_down` wacht hij de vasthoud-drempel af;
komt in die tijd `btn_up` binnen, dan wordt het dimmen afgebroken (`mode: restart`)
en blijft het een gewone klik. Anders zet hij elke ~350 ms een stap, tot je loslaat
of de lamp op 100% / de minimale helderheid zit.

## Dynamische 404-pagina

[`www/404.html`](www/404.html) is een voorbeeld van een 404-pagina die zich aanpast aan de bezoeker:

- toont het pad dat werd opgevraagd (veilig via `textContent`);
- stelt tot 3 bestaande pagina's voor op basis van gelijkenis (Levenshtein-afstand),
  bijv. `/energie` → *Energie* (`/energy`), `/logbok` → *Logboek*;
- begroeting op basis van het tijdstip en een willekeurige knipoog;
- licht/donker thema volgens het systeem;
- stuurt na 15 seconden door naar het dashboard (annuleerbaar, stopt bij interactie).

Pas bovenaan het script `KNOWN_PAGES`, `REDIRECT_TO` en `REDIRECT_AFTER_SECONDS` aan.

**Gebruiken**

Home Assistant zelf laat geen eigen 404-pagina toe, maar via een reverse proxy wel.
Kopieer het bestand naar bijv. `/var/www/errors/404.html` en voeg in nginx toe:

```nginx
server {
    # ... bestaande Home Assistant proxy-config ...
    proxy_intercept_errors on;
    error_page 404 /404.html;

    location = /404.html {
        root /var/www/errors;
        internal;
    }
}
```

De URL in de adresbalk blijft het oorspronkelijke (foute) pad, dus de pagina kan dat uitlezen.
Wil je hem alleen bekijken? Zet hem in `config/www/` en open `http://<ha>:8123/local/404.html`.

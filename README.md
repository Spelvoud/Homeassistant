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

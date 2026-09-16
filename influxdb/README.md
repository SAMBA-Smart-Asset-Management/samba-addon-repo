# InfluxDB

Vastgezette kopie van de community-add-on `hassio-addons/addon-influxdb`, zodat SAMBA-sites
InfluxDB kunnen blijven installeren.

## Waarom deze map bestaat

Op 28 augustus 2026 is de InfluxDB-add-on uit de hassio-addons-store gehaald en is
`addon-influxdb` gearchiveerd, omdat hij op InfluxDB 1.x draait en dat end-of-life is.
Gevolg: op bestaande sites draait de add-on nog wel, maar gemarkeerd als *detached*, en op
een **nieuwe site is hij niet meer te installeren**. Alle SAMBA-modules schrijven naar
InfluxDB en het Inzicht-tabblad van de app leest eruit, dus zonder deze map kan een nieuwe
site niet worden opgeleverd.

## Wat hier gebeurt

Er wordt niets gebouwd. Deze map bevat alleen de add-on-definitie; het image staat nog
publiek op GHCR en wordt gepind op de laatste gepubliceerde versie:

```
ghcr.io/hassio-addons/influxdb/{arch}:5.0.2
```

Gecontroleerd op 16 september 2026: `5.0.2` geeft anoniem een manifest (amd64 en aarch64),
`latest` bestaat niet meer. Daarom staat de versie vast en komen er geen upstream-updates
meer. Willen we ooit echt weg van 1.x, dan is dat een migratie naar VictoriaMetrics of
InfluxDB 3, geen update van deze map.

## Let op bij installatie

- **De hostname volgt de slug van dit repository.** Deze add-on wordt
  `a567d510-influxdb`, niet `a0d7b954-influxdb` zoals op sites die hem nog uit de oude
  store hebben. Zet dus per site `influxdb_url` in samba-main en het `influxdb:`-blok in
  `configuration.yaml` op de juiste host.
- **De database wordt niet automatisch aangemaakt.** Maak na de eerste start database
  `Samba` (met hoofdletter S) aan, met user `samba`. Zonder die database faalt elke write
  met een 404 en blijft het Inzicht-tabblad leeg.
- De opties hierboven zijn de upstream-standaarden. Op de bestaande SAMBA-sites draait de
  add-on met `auth: false`; dat is een per-site-optie en bewust niet in deze definitie
  veranderd.
- Verwijder op bestaande sites de reparatiemelding over de *detached* add-on niet door hem
  op te lossen: de enige actie die eraan hangt verwijdert de add-on inclusief datamap, en
  dat is de hele `Samba`-database.

## Herkomst

Oorspronkelijk werk van de Home Assistant Community Add-ons (MIT). Deze definitie is
overgenomen uit de laatste commit vóór verwijdering en aangepast: `codenotary` eruit
(ondertekeningsketen van de gearchiveerde upstream), `armv7` eruit, `url` naar dit
repository.

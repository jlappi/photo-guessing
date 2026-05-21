# Power Apps — Informationshantering
## Kravspecifikation

### Syfte

En Power Apps Canvas-app för att hantera och underhålla **Huvudlista** och dess
referenslistor. Appen ersätter direkt redigering av SharePoint-listor och möjliggör
kaskaderade uppslagsfält (cascading lookups) som SharePoint inte stöder inbyggt.

---

## Datakällor (SharePoint-listor)

| Lista | Syfte |
|---|---|
| Informationslager | Förvaringsplatser för information |
| Informationsmaterial | Materialtyper, kopplade till ett Informationslager |
| Informationsgrupp | Grupper, kopplade till ett Informationsmaterial |
| Huvudlista | Huvudlistan som byggs upp och underhålls |
| Verksamhetsprocesser | Verksamhetsprocesser och faser |
| IT-system | IT-system som används i verksamheten |
| Ansvariga enheter | Referenslista för ansvariga enheter (används i lookups) |

---

## Kaskadkedja (Cascade chain)

```
Användaren väljer → Informationsgrupp
                        └─ lookup → Informationsmaterial   (auto-ifyllt, låst)
                                        └─ lookup → Informationslager   (auto-ifyllt, låst)
```

När en användare väljer en **Informationsgrupp** i Huvudlista hämtas
**Informationsmaterial** och **Informationslager** automatiskt via lookup-kedjan
och visas som skrivskyddade fält. Om det härledda värdet ser fel ut är rätt åtgärd
att rätta källdatan i Informationsgrupp- eller Informationsmaterial-listan — inte
att skriva över värdet i Huvudlista.

---

## Skärmar (6 st)

### Skärm 1 — Huvudlista: Bläddra

**Syfte:** Visa och söka bland alla poster i Huvudlista.

**Kontroller:**
- Galleri med poster (Delprocess, Informationsgrupp, Informationsmaterial, Informationslager, Informationssystem)
- Sökfält — filtrerar på Delprocess, Informationsgrupp, Informationsmaterial, Informationslager
- Knapp **Ny post** → navigerar till Skärm 2 i nytt läge
- Tryck på en post → navigerar till Skärm 2 i redigeringsläge
- Navigeringsfält längst ned (gemensamt för alla skärmar)

---

### Skärm 2 — Huvudlista: Redigera / Ny post

**Syfte:** Skapa eller redigera en post i Huvudlista med kaskadlogik.

**Fält:**

| # | Fält | Kontrolltyp | Källa / beteende |
|---|---|---|---|
| 1 | Delprocess eller fas | Dropdown | Verksamhetsprocesser, sorterat |
| 2 | Informationsgrupp | Dropdown | Informationsgrupp, sorterat |
| 3 | Informationsmaterial | Etikett (skrivskyddad) | Auto-ifyllt från vald Informationsgrupp |
| 4 | Informationslager | Etikett (skrivskyddad) | Auto-ifyllt från Informationsmaterial |
| 5 | Informationssystem | Dropdown | IT-system, sorterat |
| 6 | Kommentar | Textfält (flerradigt) | Fri text |
| 7 | Inkluderas i beskrivning av handlingars offentlighet | Växlingsknapp (Toggle) | Ja / Nej |

Fälten 3 och 4 visas med en tydligt annorlunda bakgrundsfärg för att markera att
de är härledda och inte kan redigeras direkt.

**Power Fx — härledda fält:**

```powerfx
// Informationsmaterial — Text-egenskapen på etiketten
LookUp(
    Informationsgrupp,
    Title = ddInformationsgrupp.Selected.Title,
    Informationsmaterial.Value
)

// Informationslager — Text-egenskapen på etiketten
LookUp(
    Informationsmaterial,
    Title = lblInformationsmaterial.Text,
    'Informationsmaterialets informationslager'.Value
)
```

**Power Fx — Spara-knappen (OnSelect):**

```powerfx
Patch(
    Huvudlista,
    If(
        varEditMode,
        LookUp(Huvudlista, ID = varCurrentRecordID),
        Defaults(Huvudlista)
    ),
    {
        'Delprocess eller fas': {
            Id: ddDelprocess.Selected.ID,
            Value: ddDelprocess.Selected.Title
        },
        Informationsgrupp: {
            Id: ddInformationsgrupp.Selected.ID,
            Value: ddInformationsgrupp.Selected.Title
        },
        Informationsmaterial: {
            Id: LookUp(
                    Informationsgrupp,
                    Title = ddInformationsgrupp.Selected.Title,
                    Informationsmaterial.Id
                ),
            Value: lblInformationsmaterial.Text
        },
        Informationslager: {
            Id: LookUp(
                    Informationsmaterial,
                    Title = lblInformationsmaterial.Text,
                    'Informationsmaterialets informationslager'.Id
                ),
            Value: lblInformationslager.Text
        },
        Informationssystem: {
            Id: ddInformationssystem.Selected.ID,
            Value: ddInformationssystem.Selected.Title
        },
        Kommentar: txtKommentar.Text,
        'Inkluderas i beskrivning av handlingars offentlighet': tglInkluderas.Value
    }
);
Navigate(scrHuvudlistaBrowse, ScreenTransition.Fade)
```

**Power Fx — Ta bort-knappen (OnSelect):**

```powerfx
Remove(Huvudlista, LookUp(Huvudlista, ID = varCurrentRecordID));
Navigate(scrHuvudlistaBrowse, ScreenTransition.Fade)
```

---

### Skärm 3 — Informationsgrupp: Hantera

**Syfte:** Lägg till, redigera och ta bort Informationsgrupp-poster.

**Layout:** Galleri till vänster + redigeringsformulär till höger (eller navigering till separat redigeringsskärm).

**Fält i formuläret:**

| Fält | Kontrolltyp | Notering |
|---|---|---|
| Title | Textfält | Obligatoriskt |
| Informationsmaterial | Dropdown | Källa: Informationsmaterial |
| Typ av information | Dropdown | Choice-värden från listan |
| Beskriving av informationsgruppen | Textfält (flerradigt) | |
| Innehåller personuppgifter | Dropdown | Choice: Ja / Nej |
| Förvaringstid | Dropdown | Choice-värden från listan |
| Offentlighetsklass | Dropdown | Choice-värden från listan |
| Kommentar | Textfält | |

---

### Skärm 4 — Informationsmaterial: Hantera

**Syfte:** Lägg till, redigera och ta bort Informationsmaterial-poster.

**Fält i formuläret:**

| Fält | Kontrolltyp | Notering |
|---|---|---|
| Title | Textfält | Obligatoriskt |
| Informationsmaterialets informationslager | Dropdown | Källa: Informationslager |
| Förvaringstid | Dropdown | Choice-värden från listan |
| Förvaringskommentar | Textfält | |
| Kommentar | Textfält | |

---

### Skärm 5 — Informationslager: Hantera

**Syfte:** Lägg till, redigera och ta bort Informationslager-poster. Används sällan men
möjligheten ska finnas.

**Fält i formuläret:**

| Fält | Kontrolltyp | Notering |
|---|---|---|
| Namnet av informationslager | Textfält | Obligatoriskt |
| Ansvarig enhet | Dropdown | Källa: Ansvariga enheter |
| Kommentar | Textfält | |
| Informationslagrets användningsändamål | Textfält (flerradigt) | |
| Allmän beskrivning av utlämnande av uppgifter | Textfält (flerradigt) | |

---

### Skärm 6 — Verksamhetsprocesser: Hantera

**Syfte:** Lägg till, redigera och ta bort processer som används i Huvudlista.

**Fält i formuläret:**

| Fält | Kontrolltyp | Notering |
|---|---|---|
| Verksamhetsområde | Dropdown | Choice-värden från listan |
| Delområde | Textfält | |
| Namn av verksamhetsprocess eller fas | Textfält | Obligatoriskt, används i lookup |
| Syftet med verksamhetsprocessen | Textfält (flerradigt) | |
| Ordningsföljd | Nummerfält | |
| Ägare | Dropdown | Källa: Ansvariga enheter |

---

## Navigering

Navigeringsfält längst ned på alla skärmar med följande ikoner/etiketter:

| Ikon | Destination |
|---|---|
| Huvudlista | Skärm 1 |
| Informationsgrupp | Skärm 3 |
| Informationsmaterial | Skärm 4 |
| Informationslager | Skärm 5 |
| Processer | Skärm 6 |

---

## Uppkoppling mot SharePoint (setup-steg)

1. Öppna **Power Apps Studio** på make.powerapps.com
2. Skapa en ny **Canvas-app** (tom, telefon- eller surfplattelayout)
3. Välj **Datakällor → Lägg till datakälla → SharePoint**
4. Ange SharePoint-webbplatsens URL
5. Lägg till följande listor:
   - Informationslager
   - Informationsmaterial
   - Informationsgrupp
   - Huvudlista
   - Verksamhetsprocesser
   - IT-system
   - Ansvariga enheter
6. Kontrollera att appens tjänstkonto (eller användarna) har läs- och skrivbehörighet på alla listor

---

## Designbeslut och motiveringar

| Beslut | Motivering |
|---|---|
| Informationsmaterial och Informationslager är skrivskyddade i Huvudlista | De är härledda värden, inte användarval. Fel värde = fel källdata, åtgärdas i källlistan. |
| Informationslager är redigerbart | Listorna är i princip stabila men ett sällsynt behov kan uppstå. |
| Delområde ingår inte i Huvudlista-formuläret | Förvirrar strukturen, hanteras i Verksamhetsprocesser-listan i stället. |
| Appspråk: Svenska | Alla etiketter, knappar och meddelanden på svenska. |
| Ingen ny post i Informationslager från Informationsmaterial-skärmen | Skapas separat på Informationslager-skärmen för att hålla formulären enkla. |

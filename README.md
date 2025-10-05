# 🧭 Playwright Locator Guide

Ein vollständiger Guide für den Aufbau und die Priorisierung von **Locators** in Playwright.

---

## 🔝 Locator-Priorität (von „beste“ zu „schlechteste“)

| Priorität | Typ | Beschreibung |
|------------|------|---------------|
| 🥇 1 | `data-testid` / `data-qa` / `data-test` | Stabil, semantisch sinnvoll, unabhängig von Design |
| 🥈 2 | `id` | Schnell, eindeutig, aber oft dynamisch generiert |
| 🥉 3 | Kombination aus Attributen (z. B. `.user-card [data-testid="edit"]`) | Gut für Komponenten mit stabilem Aufbau |
| 4️⃣ | `class` | Nur nutzen, wenn eindeutig und stabil |
| 5️⃣ | Text | Gut lesbar, aber anfällig bei Sprachänderungen |
| 🚫 6 | XPath | Vermeiden – instabil, unlesbar, schwer zu pflegen |

---

## 🧩 1. Lokalisieren über ID

Wenn ein Element eine eindeutige **id** besitzt – **immer diese verwenden**.

```html
<button id="submit-btn">Absenden</button>
```

```ts
await page.locator('#submit-btn').click();
```

✅ **Best Practice**
- IDs bevorzugen, wenn sie stabil sind  
- Kein XPath, kein Text nötig  
- Schnell und eindeutig

⚠️ **Anti-Pattern**
```ts
// Zu allgemein, kann sich ändern
await page.locator('button').click();
```

---

## 🎯 2. Lokalisieren über Class

Class sollte nur verwendet werden, wenn keine ID oder Data-Attribute vorhanden sind.

```html
<button class="btn btn-primary submit-btn">Absenden</button>
```

```ts
await page.locator('.submit-btn').click();
```

✅ **Best Practice**
- Verwende eindeutige Klassen  
- Kombiniere Klassen sparsam  
- Bei CSS-Frameworks generische Klassen meiden (`.btn`, `.text-center` etc.)

⚠️ **Anti-Pattern**
```ts
await page.locator('.btn').click(); // Sehr unspezifisch
```

---

## 🧾 3. Lokalisieren über Data-Attribute

Das ist der **Goldstandard** für Testautomatisierung, da diese stabil, lesbar und unabhängig vom Styling sind.

```html
<button data-testid="submit-order">Bestellung abschicken</button>
```

```ts
await page.locator('[data-testid="submit-order"]').click();
```

✅ **Best Practice**
- Verwende `data-testid`, `data-test` oder `data-qa`
- Keine IDs oder Klassen „missbrauchen“
- Einheitlicher Namensstil im Projekt (`data-testid="user-login-button"`)

---

## 💬 4. Lokalisieren über Text

Ideal für Buttons, Labels und Links mit statischem Text.

```html
<button>Speichern</button>
```

```ts
await page.locator('text=Speichern').click();
```

Oder mit partiellem Match:

```ts
await page.locator('button:has-text("Speichern")').click();
```

✅ **Best Practice**
- Ideal für statische Texte  
- Schnell einsetzbar, gut lesbar

⚠️ **Achtung**
- Sprachabhängig (Internationalisierung!)  
- Änderungen am Text → Tests brechen

---

## 🔗 5. Kombinierte Selektoren

Kombinierte Selektoren erhöhen die Präzision, ohne unlesbar zu werden.

```html
<div class="user-card">
  <button data-testid="edit-user">Bearbeiten</button>
</div>
```

```ts
await page.locator('.user-card [data-testid="edit-user"]').click();
```

Alternativ mit Hierarchie:

```ts
await page.locator('.user-card').locator('button:has-text("Bearbeiten")').click();
```

---

## 🧩 6. Dynamische Locators (Variablen)

Wenn du Variablen einbauen willst, nutze **Template Literals**:

```ts
const username = 'Patrick';
await page.locator(`[data-testid="user-${username}"]`).click();
```

Auch für Texte:

```ts
const productName = 'SuperLaptop';
await page.locator(`text=${productName}`).click();
```

Oder kombiniert:

```ts
await page.locator(`div:has-text("${username}")`);
```

✅ **Pro-Tipp**
- Immer `${}`-Interpolation verwenden, nicht `+`-Verkettung

---

## 🧱 7. Erweiterte Selektoren: `:has()`, `:has-text()`, `nth()`

Playwright unterstützt mächtige CSS-Pseudo-Selektoren:

```ts
await page.locator('div:has(button:has-text("Speichern"))');
```

Weitere Beispiele:

```ts
// Erstes Element
await page.locator('.list-item').nth(0).click();

// Element innerhalb eines bestimmten Containers
await page.locator('.modal:has-text("Einstellungen") >> button:has-text("Speichern")').click();
```

---

## 🧩 8. Lokatoren für verschachtelte Strukturen

Wenn du innerhalb komplexer DOM-Strukturen arbeitest, kannst du „Chaining“ nutzen:

```ts
const card = page.locator('.product-card').filter({ hasText: 'Laptop' });
await card.locator('button:has-text("Kaufen")').click();
```

Oder über **role-based locators**:

```ts
await page.getByRole('button', { name: 'Absenden' }).click();
```

✅ **Best Practice**
- Verwende `getByRole` für Barrierefreiheit-konforme Elemente  
- Nutze `.filter()` bei wiederkehrenden Strukturen (Listen, Tabellen)

---

## ⚙️ 9. Debugging & Locator-Strategien in der Praxis

💡 **Locator-Playground**:  
Führe `npx playwright codegen <url>` aus, um automatisch Locatoren generieren zu lassen.

💡 **Selector Preview**:  
Mit `page.locator('selector').highlight()` kannst du direkt im Browser das Ziel prüfen.

💡 **Test-Linter**:  
Nutze ESLint-Regeln oder Pre-Commit-Hooks, um ungültige oder zu breite Selektoren zu verhindern.

---

## ❌ Vermeide folgende Anti-Patterns

| Anti-Pattern | Warum schlecht |
|---------------|----------------|
| XPath (`//div[3]/button`) | Instabil, schwer zu warten |
| `.class:nth-child(2)` | Bricht bei DOM-Änderungen |
| `button` ohne Attribut | Zu allgemein |
| `text=Speichern` bei dynamischem Text | Sprachabhängig |
| Kombination aus 4+ verschachtelten Selektoren | Schwer zu lesen und zu debuggen |

---

## ✅ Zusammenfassung

**Locator-Strategie von oben nach unten:**

1. `data-testid` oder `data-qa`
2. `id`
3. Kombination (z. B. `.card [data-testid="edit"]`)
4. `class`
5. `text`
6. Kein XPath

---

> **Tipp:**  
> Erstelle in eurem Projekt einen gemeinsamen `Locator Guide` oder eine zentrale `selectors.ts` Datei mit gut benannten, wiederverwendbaren Locatoren.

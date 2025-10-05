# 🧭 Locator Guide – Playwright Best Practices

Ein stabiler und wartbarer Test steht und fällt mit sauberen Locatoren.  
Dieser Guide zeigt dir, **wie du Webelemente in Playwright zuverlässig findest**, wie du mit dynamischen Attributen umgehst, und welche Strategien du für verschiedene HTML-Strukturen verwenden solltest.

---

## ⚙️ Grundlagen

In Playwright werden Elemente über **Locator-APIs** angesprochen.  
Beispiele:

```ts
// Beispiel in TypeScript
const button = page.locator('button#submit');
await button.click();

// oder kürzer:
await page.locator('text=Speichern').click();
Playwright verwendet "Smart Locators", d. h. es kombiniert intern Strategien, um Elemente zu finden.
Trotzdem solltest du bewusst und eindeutig schreiben, um Flakiness zu vermeiden.

🧩 1. Lokalisieren über id
Wenn ein Element eine eindeutige id besitzt – immer diese verwenden.

html
Code kopieren
<button id="submit-btn">Absenden</button>
ts
Code kopieren
await page.locator('#submit-btn').click();
✅ Best Practice:

Immer IDs bevorzugen, wenn sie stabil sind.

Kein XPath, kein Text nötig.

Schnell und eindeutig.

⚠️ Anti-Pattern:

ts
Code kopieren
// Zu allgemein, kann sich ändern:
await page.locator('button').click();
🎯 2. Lokalisieren über class
class sollte nur verwendet werden, wenn keine ID oder Data-Attribute vorhanden sind.

html
Code kopieren
<button class="btn btn-primary submit-btn">Absenden</button>
ts
Code kopieren
await page.locator('.submit-btn').click();
✅ Best Practice:

Verwende eindeutige Klassen.

Kombiniere Klassen sparsam:

ts
Code kopieren
await page.locator('button.btn-primary.submit-btn');
Wenn du CSS-Bibliotheken wie Bootstrap oder Tailwind nutzt, vermeide generische Klassen wie .btn oder .text-center.

⚠️ Anti-Pattern:

ts
Code kopieren
// Sehr unspezifisch
await page.locator('.btn').click();
🧾 3. Lokalisieren über data-* Attribute
Das ist der Goldstandard für Testautomatisierung, da diese Attribute stabil, lesbar und unabhängig vom Styling sind.

html
Code kopieren
<button data-testid="submit-order">Bestellung abschicken</button>
ts
Code kopieren
await page.locator('[data-testid="submit-order"]').click();
✅ Best Practice:

Verwende data-testid, data-test oder data-qa Attribute.

Keine IDs oder Klassen „missbrauchen“.

Konsistenter Namensstil im Projekt:

html
Code kopieren
data-testid="user-login-button"
💬 4. Lokalisieren über Text
Für Buttons, Labels und Links oft sehr praktisch.

html
Code kopieren
<button>Speichern</button>
ts
Code kopieren
await page.locator('text=Speichern').click();
Oder mit partiellem Match:

ts
Code kopieren
await page.locator('button:has-text("Speichern")').click();
✅ Best Practice:

Ideal für statische Texte.

Schnell einsetzbar, gut lesbar.

⚠️ Achtung:

Sprachabhängig (Internationalisierung!).

Änderung des Textes → Test bricht.

🔗 5. Lokalisieren über kombinierte Selektoren
Du kannst Selektoren kombinieren, um präziser zu werden:

html
Code kopieren
<div class="user-card">
  <button data-testid="edit-user">Bearbeiten</button>
</div>
ts
Code kopieren
await page.locator('.user-card >> [data-testid="edit-user"]').click();
Oder mit CSS-Notation:

ts
Code kopieren
await page.locator('.user-card [data-testid="edit-user"]').click();
Oder mit hierarchischen Bedingungen:

ts
Code kopieren
await page.locator('.user-card').locator('button:has-text("Bearbeiten")').click();
🧩 6. Lokalisieren mit Variablen / dynamischen Werten
Wenn du Variablen in Locatoren einbauen willst, nutze Template Literals (Backticks):

ts
Code kopieren
const username = 'Patrick';
await page.locator(`[data-testid="user-${username}"]`).click();
Oder dynamische Texte:

ts
Code kopieren
const productName = 'SuperLaptop';
await page.locator(`text=${productName}`).click();
Auch kombinierbar mit has-text:

ts
Code kopieren
await page.locator(`div:has-text("${username}")`);
✅ Pro-Tipp:
Immer String-Interpolation (${}) verwenden, nicht +-Verkettung.

🧱 7. :has(), :has-text(), :nth() und komplexe CSS-Logik
Playwright unterstützt CSS-Pseudo-Selektoren wie:

ts
Code kopieren
await page.locator('div:has(button:has-text("Speichern"))');
Das erlaubt sehr präzise, aber trotzdem lesbare Selektoren.

Weitere Beispiele:

ts
Code kopieren
// Erstes Element:
await page.locator('.list-item').nth(0).click();

// Element innerhalb eines bestimmten Containers:
await page.locator('.modal:has-text("Einstellungen") >> button:has-text("Speichern")');
⚡ 8. getBy-APIs (seit Playwright v1.28+)
Playwright bietet spezielle Methoden für häufige Szenarien:

ts
Code kopieren
await page.getByRole('button', { name: 'Speichern' }).click();
await page.getByLabel('Benutzername').fill('Patrick');
await page.getByPlaceholder('Passwort').fill('123456');
await page.getByText('Willkommen').isVisible();
✅ Empfohlene APIs:

getByRole() → für Buttons, Links, Checkboxen (zugänglichkeitsfreundlich)

getByLabel() → für Formulare

getByText() → für sichtbare Texte

getByTestId() → für eigene data-testid Attribute

📘 Beispiel:

ts
Code kopieren
await page.getByTestId('submit-order').click();
🧮 9. XPath (nur wenn unbedingt nötig)
XPath ist mächtig, aber fehleranfällig und schlecht wartbar.
Verwende ihn nur, wenn keine andere Option besteht (z. B. bei dynamisch generierten PDFs oder altmodischem HTML).

ts
Code kopieren
await page.locator('//div[@class="container"]//button[text()="Löschen"]');
❌ Nur letzte Option – CSS, Data-Attributes oder getBy...() sind fast immer besser.

🧰 10. Dynamische Inhalte & Wartezeiten
Wenn ein Element asynchron erscheint:

ts
Code kopieren
await page.locator('[data-testid="result"]').waitFor();
await page.locator('[data-testid="result"]').isVisible();
Oder:

ts
Code kopieren
await expect(page.locator('[data-testid="result"]')).toBeVisible();
🧠 11. Debugging & Tipps
npx playwright codegen <URL> → interaktiver Recorder mit Locator-Vorschlägen

page.pause() → Interaktives Debugging

locator.first(), locator.last(), locator.nth() → präzise Auswahl bei mehrfachen Treffern

locator.filter() → Filtert nach Text, Rolle, Sichtbarkeit

ts
Code kopieren
await page.locator('button').filter({ hasText: 'Löschen' }).click();
🧭 Locator-Priorität (von best bis worst)
Priorität	Methode	Beispiel	Stabilität	Empfehlung
1️⃣	getByTestId() / data-testid	getByTestId('submit') / [data-testid="submit"]	🟢 Sehr stabil	✅ Immer bevorzugen
2️⃣	id	#username	🟢 Stabil	✅ Gut
3️⃣	getByRole() / getByLabel()	getByRole('button', { name: 'Speichern' })	🟢 Stabil	✅ Gut
4️⃣	class	.submit-btn	🟠 Mittel	⚠️ Nur wenn eindeutig
5️⃣	Text	text=Speichern	🟠 Mittel	⚠️ Sprachabhängig
6️⃣	XPath	//button[text()="Speichern"]	🔴 Fragil	🚫 Nur Notlösung

🧩 Beispiel: Dynamischer Locator mit Variable und Fallback
ts
Code kopieren
async function clickUser(userName: string) {
  const userLocator = page.locator(`[data-testid="user-${userName}"]`);
  
  if (await userLocator.count() > 0) {
    await userLocator.click();
  } else {
    // Fallback über Text
    await page.locator(`text=${userName}`).click();
  }
}
✅ Fazit
Saubere Locatoren = stabile Tests.

Bevor du einen Locator baust, frag dich:

Gibt es ein data-testid? → Verwende es.

Gibt es eine id? → Nutze sie.

Kein Identifier? → Erstelle ein semantisches Attribut oder fallback auf getByRole() oder Text.

Weniger Magie, mehr Struktur = langfristig wartbare Tests.

yaml
Code kopieren

---

✅ **So fügst du’s ein:**
1. Öffne dein GitHub-Repo.  
2. Gehe auf `README.md`.  
3. Klick auf **Edit (✏️)**.  
4. **Markiere alles**, füge den obigen Block ein.  
5. Committen → fertig.

---

Möchtest du, dass ich dir im Anschluss noch eine zweite Datei (`LOCATOR_BEST_PRACTICES.md`) baue, die du separat verlinken kannst – z. B. mit praktischen Codebeispielen aus Playwright Page Objects (mit `this.usernameInput = page.getByTestId(...)` etc.)?  
Dann wäre dein Repo-Doku-Setup komplett rund.







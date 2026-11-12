# PV Auto – aplicație Android pentru PV-uri de predare-primire autovehicule

Aplicație pentru telefon (Android 7+) cu care se completează pe loc:

* **PV predare-primire – utilizare individuală** (după `PV_predare_primire_autovehicule_092025.docx`)
* **PV predare-primire – utilizare temporară / pool car** (după `FPVPP_poolcar_EGB/THG_rev052025.xlsx`), cu predare și, ulterior, **returnare** în același PV

La final se generează un **PDF** care se trimite pe e-mail către mai mulți destinatari.

## Ce face

| Funcție | Detalii |
|---|---|
| Secțiuni pas cu pas | Fiecare secțiune are **„Sari peste”**. În PDF apare „Secțiune necompletată – sărită la completare”. |
| Informări + GDPR | **Nu se pot sări**. „Continuă” e activ doar după bifarea „Am citit, am înțeles și iau la cunoștință”. În PDF apar numele și momentul luării la cunoștință. |
| Fotografii | Cadre recomandate (față, spate, laterale, 3/4, bord/kilometraj, interior, portbagaj, VIN) plus oricâte poze suplimentare. Fiecare poză primește automat ștampila cu data, ora, nr. auto și nr. PV. |
| Avarii | Schiță auto atingibilă: atingi locul, elementul se recunoaște automat (ușă, aripă, bară…), alegi tipul daunei și modul de reparație și poți fotografia avaria. La returnarea pool car, avariile de la predare apar cu gri, iar cele noi cu roșu. |
| Elemente din raportul de leasing (B240EGB) | Verificări funcționale (AC, faruri, geamuri, servodirecție, centuri, martori bord, zgomote motor), număr de chei, anvelope (profil pe fiecare roată, producător, dimensiuni, sezon, jante, roată de rezervă), carte de service, tip daună și mod de reparație. |
| Semnături | Predator și primitor semnează pe ecran. |
| E-mail | Se deschide Outlook/Gmail cu destinatarii completați și PDF-ul atașat, iar utilizatorul apasă „Trimite”. Destinatarii impliciți se setează pe fiecare tip de PV, iar primitorul poate primi automat o copie. |
| Fără internet | Totul se stochează local pe telefon (ciorne, istoric, PDF-uri în *Documente/PV*). Internetul e necesar doar pentru trimiterea e-mailului. |
| Societăți | Electroglobal SA (cu logo) și Tehnoglobal Era SA. Datele, logo-ul și numerotarea se pot modifica din Setări. |
| Flotă | Lista de vehicule se poate importa din Excel (copy-paste), iar la completare datele mașinii se precompletează. |

## Generare APK gratuită în cloud (GitHub Actions) – fără nimic instalat pe PC

1. Creează cont gratuit pe https://github.com, apoi **New repository** → nume `pv-auto`, bifează **Private** → *Create repository*.
2. Pe pagina repo-ului: **uploading an existing file** → trage în fereastră **tot conținutul** folderului `pentru GitHub (de incarcat)` (inclusiv folderul `.github`) → *Commit changes*.
   - Dacă folderul `.github` nu se încarcă: **Add file › Create new file**, nume `.github/workflows/build-apk.yml`, lipește conținutul fișierului → *Commit*.
3. Tabul **Actions** → rularea „Build APK” pornește singură (~5–8 min). Când apare bifa verde, deschide rularea → secțiunea **Artifacts**:
   - `PV-Auto-apk` → arhivă cu **PV-Auto.apk** (de instalat pe telefoane);
   - `CHEIE-SEMNARE-pastrati-o` (doar la prima rulare) → cheia de semnare.
4. **O singură dată, după prima rulare:** Settings › Secrets and variables › Actions › *New repository secret*:
   - `KEYSTORE_BASE64` = conținutul fișierului `KEYSTORE_BASE64.txt` din arhiva cheii;
   - `KEYSTORE_PASSWORD` = conținutul fișierului `KEYSTORE_PASSWORD.txt`.
   Astfel toate versiunile viitoare au aceeași semnătură și se instalează peste cea veche. Păstrați arhiva cheii și offline.
5. Modificări ulterioare (ex. texte în `src/forms.js`): editezi fișierul direct pe GitHub (creionul ✎) → *Commit* → APK nou în Actions.

## Alternativ: generare APK local (IT)

Cerințe: Windows, Node.js 18 sau mai nou, internet. JDK-ul și Android SDK-ul se descarcă automat, portabil, în `..\android-toolchain` (fără instalare și fără drepturi de admin).

```powershell
cd "Aplicatie PV-uri auto\app"
powershell -ExecutionPolicy Bypass -File scripts\build-apk.ps1
```

Rezultatul este `Aplicatie PV-uri auto\PV-Auto.apk`, care se instalează pe telefoane (sau se distribuie prin MDM / Intune).
Pe telefon trebuie permisă instalarea din „surse necunoscute” pentru aplicația folosită la deschiderea APK-ului.

**Important:** la prima rulare scriptul generează cheia de semnare `android\pvauto-release.jks` și fișierul `android\keystore.properties`. Păstrați copii de siguranță pentru ele. Actualizările viitoare trebuie semnate cu aceeași cheie, altfel nu se instalează peste versiunea existentă.

Alternativ, proiectul `android\` se poate deschide în Android Studio (Build › Generate Signed APK).

## Dezvoltare

```bash
npm install
npm run build        # compilează src/ în www/app.js
npm run serve        # test în browser: http://localhost:8080
npx cap sync android # copiază www/ în proiectul Android
```

Structură:

* `src/forms.js` – definițiile formularelor (secțiuni, liste de documente și echipamente, texte de informare și GDPR). **Textele legale se modifică aici.**
* `src/pdf.js` – generarea PDF (jsPDF, font DejaVu pentru diacritice)
* `src/main.js` – interfața (ecrane, pași, setări, trimitere)
* `src/widgets.js` – schița auto pentru avarii și semnătura
* `src/native.js` – camera, fișierele și e-mailul (Capacitor), cu variante pentru browser
* `android/` – proiectul Android nativ (Capacitor 8, minSdk 24, targetSdk 36)

La modificarea textelor sau a listelor: editați `src/forms.js`, apoi rulați din nou `scripts\build-apk.ps1` și măriți `versionCode` / `versionName` în `android/app/build.gradle`.

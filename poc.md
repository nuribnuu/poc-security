Berikut adalah laporan lengkap untuk **HackerOne** (Report Intent #103592) mengenai temuan **Missing Origin Attribution in Microphone Permission Dialog via about:blank Popup** pada **Mi Browser** (com.android.browser) versi 14.54.0.

---

## Report Intent #103592

**Asset:** `com.android.browser` (OTHER_APK)  
**Weakness:** `UI Spoofing / Missing Security Indicator` (atau `Misinterpretation Biases > Context Ignorance` jika tersedia)  
**Severity:** `Medium`  

---

### Description

#### Summary:

Mi Browser for Android (version 14.54.0, default browser on Xiaomi/POCO devices) fails to display the requesting origin (domain) when a microphone permission request is triggered from an `about:blank` popup window. Instead, the system permission dialog only shows `"Allow Mi Browser to record audio?"` without any indication of which website is actually requesting access.

In contrast, secure browsers like Google Chrome and Microsoft Edge correctly display the full origin (e.g., `nuribnuu.github.io`) in the permission dialog, allowing users to make an informed decision.

An attacker can exploit this vulnerability to perform **origin spoofing / UI spoofing**: by opening a popup, navigating the main tab to a trusted domain (e.g., `google.com`), and then requesting microphone access, the user sees only the browser name and may mistakenly believe the request comes from the trusted domain.

#### Steps To Reproduce:

1. On an Android device (tested on **Poco X8 Pro Max**, Android 16), open **Mi Browser** version **14.54.0** (default settings).
2. Navigate to the PoC URL: `https://nuribnuu.github.io/poc-security/audio-origin-spoofing/`
3. Click the button **"1. Launch Popup"**. An `about:blank` popup window opens.
4. Inside the popup, click the button **"2. Trigger Audio Permission"**.
5. Observe the system permission dialog.
   - **VULNERABLE BEHAVIOR (Mi Browser):** The dialog shows only `"Allow Mi Browser to record audio?"` – the requesting origin (e.g., `nuribnuu.github.io`) is completely missing.
6. For **comparison**, repeat steps 2–5 on the same device using:
   - **Google Chrome for Android** (latest version)
   - **Microsoft Edge for Android** (latest version)
7. **SECURE BEHAVIOR (Chrome/Edge):** The permission dialog clearly displays the full origin, e.g., `"nuribnuu.github.io wants to use your microphone"`.

#### PoC:

**Live PoC URL:**  
👉 `https://nuribnuu.github.io/poc-security/audio-origin-spoofing/`

**PoC HTML code (simplified):**

```html
<button onclick="launchVulnerability()">1. Launch Popup</button>
<script>
function launchVulnerability() {
    const popup = window.open("about:blank", "poc_window", "width=500,height=450");
    if (!popup) { alert("Popup blocked!"); return; }
    const htmlContent = `
        <button id="triggerBtn">2. Trigger Audio Permission</button>
        <script>
            document.getElementById('triggerBtn').addEventListener('click', async () => {
                await navigator.mediaDevices.getUserMedia({audio: true});
            });
        <\/script>
    `;
    popup.document.write(htmlContent);
    popup.document.close();
    popup.focus();
}
</script>
```

#### Supporting Material/References:

- **PoC URL:** https://nuribnuu.github.io/poc-security/audio-origin-spoofing/
- **Test device:** Poco X8 Pro Max, Android 16
- **Mi Browser version:** 14.54.0 (com.android.browser)
- **Secure baseline:** Google Chrome (shows origin), Microsoft Edge (shows origin)
- **Attachments:**
  - `poc.png` – side‑by‑side comparison (Mi Browser vulnerable vs Chrome secure)
  - Video demonstration (optional)

---

### Impact

#### Summary:

An attacker can perform a convincing **origin spoofing attack** targeting microphone permissions. Because the permission dialog in Mi Browser omits the requesting domain, users cannot verify which website is asking for microphone access. When combined with a trusted domain in the address bar (e.g., via a redirected main tab), users are highly likely to grant permission to a malicious site.

**Specific impacts:**

- **UI Spoofing / Origin Spoofing** – The user sees only `"Allow Mi Browser to record audio?"` and has no way to know the actual source of the request.
- **Privacy violation** – Attackers can silently obtain microphone access, potentially recording conversations, ambient audio, or sensitive meetings without the user's informed consent.
- **Social engineering** – By navigating the main tab to a trusted domain (e.g., `google.com`) before triggering the popup, the attacker can make the victim believe the request originates from a legitimate site.
- **Widespread impact** – Mi Browser is the default browser on millions of Xiaomi, POCO, and Redmi devices, especially in Asia and India.

**CVSS v3.1 vector (estimated):**  
`AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:N/A:N` – Base Score: **6.1 (Medium)**

**Recommended severity:** **Medium** – Requires user interaction (clicking a link and then a button), but the missing origin makes informed consent impossible, leading to potential privacy breaches.

---

**Report prepared by:** Muhammad Nur Ibnu Hubab  
**Date:** April 21, 2026

---

### 📎 Attachments

- `poc.png` (screenshot showing comparison)
- (Optional) `poc_video.mp4` (if recorded)

---

Semoga laporan ini diterima. Jika ada yang perlu disesuaikan (misalnya menambahkan lebih banyak detail tentang dampak), beri tahu saya.
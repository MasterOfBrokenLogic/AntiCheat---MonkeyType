# AntiCheat ---MonkeyType

# 🛡️ Monkeytype Anti-Bot Detection Ecosystem (README)

## 📌 Overview

This ecosystem introduces an **anti-bot detection system** to Monkeytype to counteract the increasing use of typing automation tools, such as Selenium-based scripts, PyAutoGUI macros, and other bot frameworks that compromise the integrity of typing tests.

Built natively into the Monkeytype frontend and paired with a backend report logger, this system ensures fair competition, user authenticity, and platform trustworthiness.

---

## ⚙️ Components

### 1. **Frontend Detection Layer**

- Integrated via a `<useEffect>` hook inside the React `App.tsx`.
- Monitors:
  - `navigator.webdriver` (Selenium-like detection)
  - Plugin presence (`navigator.plugins`)
  - User Agent anomalies (`HeadlessChrome`, `PhantomJS`, etc.)
  - Lack of natural mouse movement
  - Typing interval consistency (to detect robotic, same-delay typing)
  - Tab inactivity during test

### 2. **Bot Likelihood Evaluation**

- If **≥3 suspicious behaviors** are flagged, the user is considered likely to be a bot.
- The system:
  - Alerts the user ⚠️
  - Optionally disables or invalidates the test
  - Sends data to a secure endpoint `/api/report-bot`

### 3. **Server-Side Report Logger**

- Receives JSON reports from the frontend
- Logs reports for moderation analysis
- Enables potential user flagging, test invalidation, or real-time banning (optional)

---

## 🧠 How It Works

1. User enters the site.
2. The detection system begins tracking interaction patterns (typing + mouse).
3. After 15 seconds:
   - Flags are evaluated.
   - If suspicious, the report is sent.
   - The test may be marked as “bot-influenced.”

---

## 📁 File/Code Integration

### ➕ Frontend Code Location:

`src/App.tsx`

```tsx
useEffect(() => {
  const suspiciousFlags = {
    isWebDriver: navigator.webdriver,
    noPlugins: navigator.plugins.length === 0,
    headlessUserAgent: /HeadlessChrome|PhantomJS|puppeteer|selenium/i.test(navigator.userAgent),
    noMouseMovement: true,
    sameDelayTyping: false,
    noFocus: false,
  };

  const typingTimestamps: number[] = [];
  const MAX_TYPING_LOG = 50;

  const handleMouseMove = () => {
    suspiciousFlags.noMouseMovement = false;
  };

  const handleKeyDown = () => {
    const now = performance.now();
    typingTimestamps.push(now);
    if (typingTimestamps.length > MAX_TYPING_LOG) {
      typingTimestamps.shift();
    }

    if (typingTimestamps.length > 10) {
      const deltas = typingTimestamps.slice(1).map((t, i) => t - typingTimestamps[i]);
      const avgDelta = deltas.reduce((a, b) => a + b, 0) / deltas.length;
      const variance = deltas.reduce((a, b) => a + Math.abs(b - avgDelta), 0) / deltas.length;
      suspiciousFlags.sameDelayTyping = variance < 5;
    }
  };

  const handleBlur = () => {
    suspiciousFlags.noFocus = true;
  };

  window.addEventListener('mousemove', handleMouseMove);
  window.addEventListener('keydown', handleKeyDown);
  window.addEventListener('blur', handleBlur);

  const evaluateFlags = () => {
    const botIndicators = Object.values(suspiciousFlags).filter(Boolean).length;

    if (botIndicators >= 3) {
      alert('⚠️ Automation detected. This test may be flagged or ignored.');

      fetch('/api/report-bot', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          timestamp: new Date().toISOString(),
          flags: suspiciousFlags,
          userAgent: navigator.userAgent,
        }),
      });
    }

    // Cleanup
    window.removeEventListener('mousemove', handleMouseMove);
    window.removeEventListener('keydown', handleKeyDown);
    window.removeEventListener('blur', handleBlur);
  };

  const timeout = setTimeout(evaluateFlags, 15000);

  return () => clearTimeout(timeout);
}, []);

```

### ➕ Backend Route Example (Express.js):

```js
app.post('/api/report-bot', (req, res) => {
  const report = req.body;
  console.log('[BOT DETECTION]', report);
  // Save to DB / File / Notify Admin
  res.status(200).send('Report logged');
});
```

---

## 🧪 Testing & Debugging

- Use headless browsers, Selenium, and PyAutoGUI scripts to simulate botting behavior.
- Confirm:
  - Alerts are triggered
  - Flags are correctly set
  - Reports are received on backend

---

## 📊 Future Enhancements

- 🎯 Machine learning model to better classify typing behavior
- 📍 IP & session correlation for repeat offenders
- 🧱 CAPTCHA verification for flagged users
- 🔧 Admin dashboard to manage reports

---

## 🙌 Credits

Developed in the spirit of clean competition and community fairness. Special shoutout to [@Miodec](https://github.com/Miodec) for creating Monkeytype.

---

## 🚀 Let's Keep Monkeytype Human

With the rise of AI and bots, it's more important than ever to protect what makes Monkeytype special: **raw, human typing skill.**

> "Monkeytype was built by humans, for humans.
>
> If your WPM is real --We salute you.
>
> If it's fake --we'll catch you. Swiftly.
>
> No mercy for macros. No place for cheaters"
>
> Let then type. Let us hunt.🧠⌨️


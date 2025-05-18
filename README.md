# X AutoScroll Userscript

A compact userscript that adds a Start/Stop button to auto‑scroll your X (formerly Twitter) timeline.

---

## Installation (Tampermonkey)

1. **Install Tampermonkey**
   • Chrome / Edge: [https://tampermonkey.net/?browser=chrome](https://tampermonkey.net/?browser=chrome)
   • Firefox: [https://tampermonkey.net/?browser=firefox](https://tampermonkey.net/?browser=firefox)

2. **Add the script**

   * **Fast way:** click the **Raw** button on the script file in this repo → Tampermonkey should prompt to install.
   * **Manual way:**

     1. In Tampermonkey’s dashboard, hit **➕ Create a new script**.
     2. Delete the template, paste in `x-autoscroll.user.js` from this repo.
     3. **File → Save** (or **⌘/Ctrl‑S**).

3. **Done.** Head to [https://x.com](https://x.com), tap the blue floating button to start, red to stop.

---

### Customising

| Setting    | Where         | Default  | Notes                        |
| ---------- | ------------- | -------- | ---------------------------- |
| `INTERVAL` | top of script | `700` ms | Lower = faster scroll rate   |
| `FACTOR`   | top of script | `0.75`   | `1` = full viewport per tick |

Edit the values, save, refresh X.

---

### License

MIT


// ==UserScript==
// @name         X AutoScroll
// @namespace    https://x.com
// @version      0.4.1
// @description  Toggle‑able auto‑scroll for X.com timelines.
// @match        https://x.com/*
// @grant        none
// @license      MIT
// ==/UserScript==

(() => {
  'use strict';

  let timer = null;
  const BTN_ID = 'x-autoscroll-btn';
  const INTERVAL = 700;          // ms between scrolls
  const FACTOR   = 0.75;         // viewport multiplier

  const scroll = () => window.scrollBy(0, innerHeight * FACTOR);

  const toggle = btn => {
    timer = timer ? (clearInterval(timer), null) : setInterval(scroll, INTERVAL);
    const active = Boolean(timer);
    btn.textContent     = active ? '⏹️ Stop AutoScroll' : '▶️ Start AutoScroll';
    btn.style.background = active ? '#e0245e'          : '#1da1f2';
  };

  const spawnBtn = () => {
    const btn = Object.assign(document.createElement('button'), {
      id: BTN_ID,
      textContent: '▶️ Start AutoScroll',
      onclick() { toggle(this); }
    });

    Object.assign(btn.style, {
      position:'fixed',bottom:'20px',right:'20px',zIndex:9999,
      padding:'10px 15px',background:'#1da1f2',color:'#fff',border:'none',
      borderRadius:'25px',cursor:'pointer',fontSize:'14px',fontWeight:'bold',
      boxShadow:'0 4px 12px rgba(0,0,0,.2)',transition:'background-color .2s'
    });

    document.body.appendChild(btn);
  };

  const ensureBtn = () => {
    if (!document.getElementById(BTN_ID) && document.querySelector('[data-testid="primaryColumn"]')) {
      spawnBtn();
    }
  };

  const observer = new MutationObserver(ensureBtn);

  const init = () => {
    ensureBtn();
    observer.observe(document.body, { childList:true, subtree:true });
  };

  document.readyState === 'loading' ?
    document.addEventListener('DOMContentLoaded', init) :
    init();
})();

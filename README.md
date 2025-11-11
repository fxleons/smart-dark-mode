# smart-dark-mode
some stuff

*open source* ***universal*** dark mode

# the code is here btw

// ==UserScript==
// @name         Smart Dark Mode (Alt+\ Toggle + Popup)
// @namespace    https://https://github.com/fxleons/smart-dark-mode/
// @version      2.2
// @description  Dark mode for all sites, toggle with Alt+\, remembers per site, with popup indicator
// @match        *://*/*
// @run-at       document-start
// @grant        GM_getValue
// @grant        GM_setValue
// ==/UserScript==

(function() {
    'use strict';

    // --- Safe storage fallback (GM_* or localStorage) ---
    const safeGet = (key, def) => {
        try {
            if (typeof GM_getValue === 'function') return GM_getValue(key, def);
        } catch {}
        const v = localStorage.getItem(key);
        return v === null ? def : v === 'true';
    };
    const safeSet = (key, val) => {
        try {
            if (typeof GM_setValue === 'function') return GM_setValue(key, val);
        } catch {}
        localStorage.setItem(key, val);
    };

    // --- Styles ---
    const css = `
        :root {
            --bg: #121212;
            --bg-secondary: #1e1e1e;
            --text: #e0e0e0;
            --link: #80b3ff;
            --link-visited: #b380ff;
            --border: #333;
        }

        html.darkmode-enabled, body.darkmode-enabled {
            background-color: var(--bg) !important;
            color: var(--text) !important;
        }

        html.darkmode-enabled * {
            background-color: transparent !important;
            color: var(--text) !important;
            border-color: var(--border) !important;
        }

        html.darkmode-enabled input,
        html.darkmode-enabled textarea,
        html.darkmode-enabled select {
            background-color: var(--bg-secondary) !important;
            color: var(--text) !important;
            border: 1px solid var(--border) !important;
        }

        html.darkmode-enabled a { color: var(--link) !important; }
        html.darkmode-enabled a:visited { color: var(--link-visited) !important; }

        html.darkmode-enabled ::selection {
            background: #444 !important;
            color: #fff !important;
        }

        html.darkmode-enabled ::-webkit-scrollbar {
            width: 10px;
            height: 10px;
        }
        html.darkmode-enabled ::-webkit-scrollbar-track {
            background: var(--bg-secondary) !important;
        }
        html.darkmode-enabled ::-webkit-scrollbar-thumb {
            background: #555 !important;
            border-radius: 5px;
        }
        html.darkmode-enabled ::-webkit-scrollbar-thumb:hover {
            background: #777 !important;
        }

        /* Popup indicator */
        #darkmode-popup {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: rgba(30, 30, 30, 0.9);
            color: #fff;
            padding: 10px 16px;
            border-radius: 6px;
            font-family: sans-serif;
            font-size: 13px;
            z-index: 999999;
            opacity: 0;
            transition: opacity 0.2s ease;
            pointer-events: none;
        }
        #darkmode-popup.show { opacity: 1; }
    `;

    // --- Inject style immediately ---
    const style = document.createElement('style');
    style.textContent = css;
    document.documentElement.appendChild(style);

    const siteKey = `darkmode_${location.hostname}`;
    const enabled = safeGet(siteKey, true);
    if (enabled) document.documentElement.classList.add('darkmode-enabled');

    // --- Popup indicator ---
    function showPopup(enabled) {
        let popup = document.getElementById('darkmode-popup');
        if (!popup) {
            popup = document.createElement('div');
            popup.id = 'darkmode-popup';
            document.body.appendChild(popup);
        }
        popup.textContent = enabled ? 'dark mode on' : 'bright ass shi is back';
        popup.classList.add('show');
        setTimeout(() => popup.classList.remove('show'), 1500);
        setTimeout(() => popup.remove(), 1800); // clean up for SPA navigation
    }

    // --- Toggle with Alt + \ ---
    document.addEventListener('keydown', e => {
        if (e.altKey && e.key === '\\') {
            const isActive = document.documentElement.classList.toggle('darkmode-enabled');
            safeSet(siteKey, isActive);
            showPopup(isActive);
        }
    });
})();

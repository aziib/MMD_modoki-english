# language

This folder contains multilingual UI dictionaries.

- `ja.json`: Japanese UI dictionary
- `en.json`: English UI dictionary

`src/i18n.ts` acts as a thin wrapper that reads JSON files from this folder and passes them to `i18next`.

Usage rules:

- Add new keys to both `ja.json` and `en.json`
- Prefer DOM reflection via `data-i18n*` attributes
- For experimental text, finalize in Japanese first, then expand to English

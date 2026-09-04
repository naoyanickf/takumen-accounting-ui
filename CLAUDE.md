# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

「宅麺ツールズ」: a small internal admin UI for 宅麺 accounting operations. It is a plain static site (no bundler, no framework, no tests) originally created from the CodeSandbox `static` template. All logic is vanilla JS calling AWS Lambda Function URLs (`*.lambda-url.ap-northeast-1.on.aws`) directly from the browser. The backend lives elsewhere; this repo is the frontend only.

## Commands

```bash
yarn install        # installs `serve` (the only dependency)
yarn start          # serves the repo root with `serve` (defaults to http://localhost:5000)
```

There is no build step (`yarn build` just echoes a message), no linter, and no test suite. Verify changes by opening the pages in a browser.

## Structure

Three pages share the same Bootstrap 5 (CDN) header and nav-tabs; the nav is duplicated by hand in each HTML file, so a nav change must be applied to all three:

- `index.html` + `action.js`: top page with two actions.
  - `syncCompanies()`: GET to a Lambda that syncs Kintone 振込先 → Board 発注先. The Lambda returns a JSON array of validation errors (`company_id`, `company_name`, `error`); a non-empty array is downloaded as a CSV via `exportCSV()`. `data.json` is a saved example of that error payload and is not loaded by any page.
  - `syncPayouts()`: POST multipart (`month`, `arrivals` file) to a Lambda that merges Kintone data with the ロジザード入荷ファイル and imports 支払データ into Board. Response is `{success, error}`.
- `histories.html` + `histories.js`: fetches a Lambda that returns an array of rows and renders 日時 (`row[2]`) and ログファイルURL (`row[1]`) into `#histories-table`. Runs on page load.
- `barcode.html`: a plain `<form method="get">` that submits a 13-digit JAN code straight to a barcode-image Lambda (no JS involved; the `histories.js` include at the bottom is leftover and does nothing useful there).

`action.js` also owns the full-screen loading overlay (`startLoading`/`hideLoading` inject their own CSS and DOM on first use) and the CSV export helper (adds a UTF-8 BOM so Excel opens Japanese text correctly).

## Conventions

- Lambda URLs are hard-coded in the JS/HTML; there is no config layer or environment switching.
- UI copy, alerts, and comments are in Japanese. Keep new user-facing text in Japanese.
- Pages use absolute paths (`/`, `/histories.html`) for nav links, so the site must be served from the repo root.

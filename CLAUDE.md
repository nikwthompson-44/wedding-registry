# Wedding Registry

Simple gift registry backed by a Google Sheet, embedded via iframe in a Webflow wedding site.

## Architecture
- **Backend**: Google Apps Script deployed as web app (doGet/doPost) — lives in the Google Sheet, not this repo
- **Frontend**: Single `index.html` — no build step, no dependencies
- **Hosting**: GitHub Pages
- **Integration**: iframe embed in Webflow

## Config
The `CONFIG` object at the top of the `<script>` block in `index.html` contains:
- `SCRIPT_URL` — Google Apps Script deployment URL
- `PAYPAL_URL` — PayPal.me link for honeymoon fund

## Google Sheet columns
`Item_Name`, `Image_1`, `Image_2`, `Description`, `URL`, `Taken`

## Brand
- Primary green: #5C6E42 (headings)
- Dark green: #223E1F (buttons/CTAs)
- Background: #FEFEFE
- Headings: Cormorant Garamond
- Body: DM Sans

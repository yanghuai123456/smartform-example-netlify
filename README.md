# Netlify contact form (Netlify Forms alternative) with AI spam filtering

Two ways to add a SmartForm contact form to a Netlify-hosted site.

## What you're POSTing

The endpoint accepts a standard HTML form POST or JSON via AJAX. Two
kinds of fields:

**Your form fields** â€?`name`, `email`, `message`, whatever you
want. Every non-reserved field lands in your dashboard as a column in
the submissions table.

**Reserved fields** â€?names starting with `_` are interpreted by
the API, not stored:

| Field | Purpose |
|---|---|
| ``_gotcha`` | **Honeypot.** Keep it empty. Hidden from humans via CSS; bots fill it automatically. Any non-empty value silently drops the submission. Add this to every form. |
| ``_hp_email`` / ``_website`` / ``_url`` / ``_phone`` | Honeypot aliases for `_gotcha` (WordPress / WPForms / Contact Form 7 migrations). Same drop semantics. |
| ``_next`` | Same-origin URL to redirect to after a successful submission. Browser POST results in a 302 here. AJAX calls (with `Accept: application/json`) get the same value back as `next_url` in the JSON response. Only http(s) and in-site paths allowed. |
| ``_subject`` | Override the AI-generated email subject line. Max 200 chars; control characters stripped. |
| `X-Gotcha` header | Same as `_gotcha` for JSON requests where you can't add a hidden form field. |

Field names are Formspree-compatible â€?migrating from
`formspree.io/f/{form_id}` requires no renaming.

## Setup

1. Get a form ID at https://usesmartform.com/dashboard.
2. Clone, configure, deploy:
   ```bash
   git clone https://github.com/smartformai/smartform-example-netlify.git
   cd smartform-example-netlify
   # edit netlify.toml â†?context "production" env SMARTFORM_FORM_ID
   npx netlify deploy --prod
   ```
3. Open the deployed URL, submit, check your dashboard.

## Option A â€?pure static HTML form (no Netlify Functions)

`index.html` ships a plain HTML form that posts directly to SmartForm's public endpoint.
Netlify's static hosting serves it; there is no Netlify Function involved.

```html
<form action="https://api.usesmartform.com/api/v1/f/YOUR_FORM_ID" method="POST">
  <input  name="name"    required />
  <input  name="email"   type="email" required />
  <textarea name="message" required></textarea>
  <input  type="text" name="_gotcha" tabindex="-1" autocomplete="off"
          style="position:absolute;left:-9999px" aria-hidden="true" />
  <button type="submit">Send</button>
</form>
```

Replace `YOUR_FORM_ID` with the 8-char ID from your dashboard.

## Option B â€?Netlify Function proxy

`netlify/functions/submit.mjs` is a small serverless function that forwards browser submissions
to SmartForm. Use this when you want to:

- Keep the form ID out of the public HTML.
- Add validation or field normalization before forwarding.
- Reuse an existing Netlify Function codebase.

The browser calls `/.netlify/functions/submit` and the function does the upstream POST.

## How the API works

- `POST https://api.usesmartform.com/api/v1/f/{form_id}` â€?JSON or form-data, no API key.
- Response: `{ success, message, submission_id, is_spam, intent, next_url }`.

For the full contract, see https://usesmartform.com/docs.


## FAQ

### Is there a free tier?

Yes. AI spam filtering is enabled by default on every plan. AI intent
classification and high-value lead detection require a paid plan (Pro
or Business) â€?the dashboard enforces this and returns HTTP 402 if
you try to enable them on a free workspace.

### Do I need an API key?

No. The form posts directly to a public endpoint using only an 8-char
form ID, which is non-enumerable. The example also includes a hidden
`_gotcha` honeypot field so naive bots cannot submit.

### How is this different from Netlify Forms?
Netlify Forms is bound to Netlify hosting. This example works on any static host and adds an AI spam filter instead of honeypot-only detection.

## Related examples
[Vercel Functions proxy](https://github.com/smartformai/smartform-example-serverless-vercel) | [Cloudflare Pages contact form](https://github.com/smartformai/smartform-example-cloudflare-react) | [Astro contact form](https://github.com/smartformai/smartform-example-astro)


## License

MIT.


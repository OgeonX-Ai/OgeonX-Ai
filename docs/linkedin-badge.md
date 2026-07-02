# LinkedIn Badge Kit

Use this page as the reusable badge kit for OgeonX AI, GitHub profiles, GitHub Pages, portfolio pages and landing pages.

## 1. GitHub README badge

GitHub README files do not allow LinkedIn's JavaScript badge, so use SVG badges instead.

```md
<p align="left">
  <a href="https://www.linkedin.com/in/kimharjamaki/">
    <img src="https://img.shields.io/badge/LinkedIn-Kim%20Harjam%C3%A4ki-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn profile" />
  </a>
  <a href="https://github.com/OgeonX-Ai">
    <img src="https://img.shields.io/badge/GitHub-OgeonX--Ai-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub organization" />
  </a>
  <a href="https://github.com/OgeonX-Ai/OgeonX-Ai/blob/main/privacy.html">
    <img src="https://img.shields.io/badge/Privacy-Policy-0A7CFF?style=for-the-badge" alt="Privacy policy" />
  </a>
</p>
```

## 2. Official LinkedIn dark profile badge for websites

Paste this once before the closing `</body>` tag:

```html
<script src="https://platform.linkedin.com/badges/js/profile.js" async defer type="text/javascript"></script>
```

Then paste this where the badge should appear:

```html
<div class="badge-base LI-profile-badge"
     data-locale="en_US"
     data-size="medium"
     data-theme="dark"
     data-type="VERTICAL"
     data-vanity="kimharjamaki"
     data-version="v1">
  <a class="badge-base__link LI-simple-link" href="https://www.linkedin.com/in/kimharjamaki/">Kim H.</a>
</div>
```

## 3. Plain HTML fallback button

Use this when JavaScript is blocked or when a privacy-safe static link is preferred.

```html
<a href="https://www.linkedin.com/in/kimharjamaki/"
   style="display:inline-flex;align-items:center;gap:10px;padding:12px 16px;border:1px solid #0a7cff;border-radius:999px;background:#06090f;color:#fff;text-decoration:none;font-family:system-ui,-apple-system,Segoe UI,sans-serif;font-weight:700;">
  <span style="color:#0a7cff;">in</span>
  Connect with Kim Harjamäki on LinkedIn
</a>
```

## Recommended placement checklist

- GitHub profile README
- OgeonX AI organization profile README
- GitHub Pages landing page
- Product landing page footer
- Personal portfolio page
- Online CV page
- Blog sidebar
- Developer documentation footer
- Demo app about/contact section
- Email signature as a plain LinkedIn URL

## Notes

- Do not use the JavaScript badge inside GitHub README files. GitHub strips scripts.
- Keep LinkedIn logos out of the OAuth app icon and developer app branding.
- Use the official badge only on normal web pages where third-party scripts are acceptable.
- Use the static SVG/button fallback for privacy-sensitive pages.

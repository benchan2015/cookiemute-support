# CookieMute Support

Use [GitHub Issues](https://github.com/benchan2015/cookiemute-support/issues) for bug reports and questions.

## Before you file a report

Please include:

1. **Site URL** where the banner appeared.
2. **Browser and version** (for example, Chrome 141 on Windows 11).
3. **CookieMute version** from `chrome://extensions`.
4. **What happened** versus what you expected — for example, "the banner stayed open" or "the wrong button was clicked".
5. **Screenshot** if possible, especially of the banner.

Do not include passwords, cookie values, or other personal data.

## Common questions

**Does CookieMute send anything to a server?**
No. It makes no network requests and has no analytics.

**A banner was not rejected. What can I try?**
Reload the page after installing or updating the extension, then use **Manual Reject Now** in the toolbar popup. If it still fails, open an issue with the site URL.

**Why is the hide fallback off by default?**
Hiding consent UI can affect sites that rely on it. Enable it in the popup only if you want the fallback.

**How do I exclude a site?**
Open the popup on that site and choose **Add current site to whitelist**. Subdomains are covered automatically.

# WP Publisher

A lightweight, self-contained HTML tool for publishing and editing articles on a WordPress site — no WordPress admin required. Built for NGOs, foundations, and small organisations who want to publish content without touching WordPress admin.

Designed for non-technical users: a social worker or volunteer can open the file in any browser and publish content directly, without ever logging into WordPress or touching Elementor.

---

## What it does

- Write and publish new articles with a simple rich-text editor (bold, italic, headings, lists)
- Edit existing articles — change content, author, category, tags, featured image
- Upload featured images (drag and drop or click to browse)
- Add and remove keyword tags
- Choose author, category, and publish status (draft or live)
- Works from any computer, no installation needed

---

## How it works

The tool talks to the WordPress REST API. Because browsers block direct cross-origin requests to WordPress from local HTML files (CORS), a lightweight Cloudflare Worker acts as a proxy in between. The Worker simply forwards requests from the browser to WordPress and adds the necessary CORS headers.

```
Your browser → Cloudflare Worker → WordPress REST API
```

---

## Setup

### Step 1 — Create a Cloudflare Worker

1. Sign up for a free account at [cloudflare.com](https://cloudflare.com)
2. Go to **Workers & Pages → Create application → Start with Hello World**
3. Give it a name (e.g. `my-site-proxy`)
4. Click **Edit code**, delete the default code, and paste this:

```javascript
export default {
  async fetch(request) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
          'Access-Control-Allow-Headers': 'Content-Type, Authorization',
        }
      });
    }

    const url = new URL(request.url);
    const target = 'https://YOUR-WORDPRESS-SITE.com' + url.pathname + url.search;

    const contentType = request.headers.get('Content-Type') || '';

    const res = await fetch(target, {
      method: request.method,
      headers: {
        'Authorization': request.headers.get('Authorization') || '',
        ...(contentType && { 'Content-Type': contentType }),
      },
      body: request.method !== 'GET' ? request.body : undefined,
    });

    const data = await res.arrayBuffer();

    return new Response(data, {
      status: res.status,
      headers: {
        'Content-Type': res.headers.get('Content-Type') || 'application/json',
        'Access-Control-Allow-Origin': '*',
      }
    });
  }
};
```

5. Replace `YOUR-WORDPRESS-SITE.com` with your actual domain
6. Click **Deploy**
7. Copy the Worker URL — it will look like `https://my-site-proxy.yourname.workers.dev`

The free Cloudflare plan includes 100,000 requests per day, which is more than enough.

---

### Step 2 — Create a WordPress Application Password

Application passwords are separate from your main WordPress password and can be revoked at any time.

1. Log into WordPress admin
2. Go to **Users → Your Profile** (or the profile of the user you want the tool to publish as)
3. Scroll down to **Application Passwords**
4. Type a name (e.g. `Publisher Tool`) and click **Add New Application Password**
5. Copy the password shown — it looks like `AbCd EfGh IjKl MnOp QrSt UvWx`
   - You only see it once, so copy it now
6. Make sure this user has at least the **Author** role in WordPress
   - If you want the tool to be able to set other users as authors, the user needs **Administrator** role

---

### Step 3 — Configure the publisher tool

Open `publisher.html` in a text editor and find these three lines near the bottom inside the `<script>` tag:

```javascript
const PROXY = 'YOUR_CLOUDFLARE_WORKER_URL';
const USER  = 'YOUR_WORDPRESS_USERNAME';
const PASS  = 'YOUR_APPLICATION_PASSWORD';
```

Replace the values:

```javascript
const PROXY = 'https://my-site-proxy.yourname.workers.dev';
const USER  = 'your-wordpress-username';
const PASS  = 'AbCd EfGh IjKl MnOp QrSt UvWx';
```

Save the file. That's it.

---

### Step 4 — Use it

Open `publisher.html` in any browser (Chrome, Firefox, Safari, Edge).

- Click **Test connection** to confirm everything works
- Use the **New article** tab to write and publish
- Use the **Edit existing** tab to find and update any published post

The tool works fully offline except for the actual publish/save action, which requires internet.

---

## Sharing with a team member

Just send them the configured `publisher.html` file. They open it in their browser — no account, no installation, no WordPress login needed.

**Security note:** The application password is embedded in the file. Only share it with people you trust. If you need to revoke access, go to **WordPress admin → Users → Profile → Application Passwords** and delete the relevant password. Generate a new one and update the file for anyone who still needs access.

---

## Revoking access

To immediately cut off access for anyone who has the file:

1. Go to WordPress admin → Users → the publishing user's profile
2. Scroll to Application Passwords
3. Click **Revoke** next to the password used in the tool

Anyone with the old file will get an authentication error immediately.

---

## Requirements

- A WordPress site with the REST API enabled (enabled by default on all modern WordPress installs)
- A Cloudflare account (free tier is sufficient)
- Any modern web browser

No plugins required. No server-side code. No build process.

---

## File structure

```
publisher.html    — the entire tool, one self-contained file
README.md         — this file
```

---

## License

MIT — free to use, modify, and share.

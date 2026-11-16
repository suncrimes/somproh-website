# somproh-website

WordPress site hosted on one.com (MariaDB). This repo is a **copy/backup of the site files** — pushing here does
not change the live site. Upload changed files to one.com via SFTP / File Manager.

- WordPress core: 7.1.2 (7.1.4 available — update from wp-admin)
- Theme: Blogier (child of Blogus), Twenty Twenty-Five kept as fallback
- Plugins: Really Simple SSL, Simple Custom CSS & JS, Akismet

`wp-config.php` is **not** in the repo on purpose (it holds the database password). See `.gitignore`.

## Security changes in this version

| File | Change |
|---|---|
| `.htaccess` | Blocks `wp-config.php`, `readme.html`, `license.txt`, dotfiles (`.git`, `.env`), backup/SQL dumps, `xmlrpc.php`, `?author=` user enumeration; adds security headers (HSTS, nosniff, frame-options, referrer, permissions) |
| `wp-content/uploads/.htaccess` | Blocks execution of PHP/scripts in uploads (Apache 2.4 syntax + 2.2 fallback) |
| `wp-content/uploads/custom-css-js/51.js` | Fixed JS syntax error (`elstyle` → `el.style`, missing `);`) — the script never ran before |
| Removed | `hello.php` (Hello Dolly), `ansar-import` (demo importer, not needed after setup), unused themes Twenty Twenty-Two/Three/Four |
| `.gitignore` | Keeps passwords, DB dumps, zips, logs and cache out of git |

## Checklist for the live server (must be done on one.com / in wp-admin)

### 1. Rotate leaked secrets — URGENT
The old `wp-config.php` (with DB password and keys) was pushed to this **public** repo in `www.zip`.
1. one.com control panel → **Database** → change the MariaDB user password → put it in `DB_PASSWORD` in the live `wp-config.php`.
2. Replace the 8 `AUTH_KEY … NONCE_SALT` lines with fresh ones from https://api.wordpress.org/secret-key/1.1/salt/ (this logs everyone out).
3. Change all WordPress admin passwords and the one.com account password. Enable 2FA (Really Simple SSL → Login Protection).
4. Check **Users** in wp-admin for admin accounts you don't recognise.

### 2. Add to the live `wp-config.php`
Paste above the line `/* That's all, stop editing! */`:

```php
define( 'DISALLOW_FILE_EDIT', true );      // no theme/plugin editor in wp-admin
define( 'WP_DEBUG', false );
define( 'WP_DEBUG_DISPLAY', false );
define( 'FORCE_SSL_ADMIN', true );
define( 'WP_AUTO_UPDATE_CORE', 'minor' );  // automatic security releases
define( 'WP_POST_REVISIONS', 10 );
```

### 3. Updates (wp-admin → Dashboard → Updates)
- WordPress core 7.1.2 → 7.1.4
- Update all plugins and themes (Akismet 5.3.3, Really Simple SSL 9.1.2, Custom CSS & JS 3.48, Blogus 2.2.1, Blogier 1.8 are all from 2024)
- Turn on auto-updates for plugins
- Delete in wp-admin: Hello Dolly, Ansar Import, Twenty Twenty-Two/Three/Four (deleting them in git does not remove them from the server)

### 4. one.com settings
- PHP version: newest available (8.3/8.4)
- Make sure the one.com SSL certificate is active
- Enable one.com backups, or set up a backup plugin
- File permissions: `wp-config.php` 600 or 640, folders 755, files 644

### 5. Custom JS fix
The site script in *Custom CSS & JS* (#51) had a syntax error. In wp-admin → Custom CSS & JS → edit #51 and replace with:

```js
document.querySelectorAll('.background-animation, .animation, .shapes, .shape-container').forEach(el => {
    el.style.display = 'none';
});
```

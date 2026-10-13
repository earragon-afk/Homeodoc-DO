# Homeodoc static website

The site has been reconstructed from the November 3, 2021 CMS Made Simple database backup and the matching images. The original archive is unchanged. This folder is independent of the surrounding Mila project.

Only `public/` is the website. It contains static HTML, CSS, one small script for legacy query links, and images. It needs no PHP, MySQL, CMS, Node.js, or Python in production. The core pages and navigation also work with JavaScript disabled.

## Included

- 58 active CMS pages, 46 published news/testimonial articles, and 30 referenced image assets.
- Original French and English copy and original desktop design.
- 117 HTML files in total: 20 linked website pages and the error page at the root, plus 96 archived pages in `public/unsused/`.
- Rebuilding keeps unused pages in `unsused/` and updates their internal links and legacy redirects.
- Repairs for the misspelled home alias and broken testimonial section links.
- Contact form replacements that open the visitor's email application, addressed to `info@homeodoc.ca`.
- Retired PayPal payment sections removed from both contact pages.
- Nginx redirects for old `index.php?page=...` links using aliases or page IDs, and `cntnt01articleid=...` news links.

The original 1000-pixel desktop design is preserved. Below 1000 pixels, the layout reflows with readable text, wrapping navigation, simplified decorative borders and larger section links. Below 600 pixels, images, maps and the contact section stack vertically. Phone widths of 320 and 390 pixels and a 768-pixel tablet layout were checked in the browser, including the contact sections and French/English navigation. Prices, contact details and editorial claims are copied from the backup, not verified or updated. Google Maps still depends on its external service.

The older loose HTML files and `vieux-site/` represent a previous site version. They remain in the source archive; the loose root HTML filenames redirect to the corresponding CMS-era pages. URLs inside `vieux-site/` are not reproduced. Hidden CMS pages, including old backups and test pages, are retained with `noindex` and outside the original main menu. CMS administration, credentials, SQL dumps, logs and private form submissions are not included in `public/`.

The original English “Treated Cases” page was empty; it now contains a full translation of the French case notes, with 24 body-system sections and quick links. English testimonials now follow the French category order with 25 quick links; missing French entries and updated passages have been translated, and existing English translations and English-only entries have been retained. Categories without case notes in the French source remain empty rather than inventing content. Translation sources live in `content/`: edit `cured-cases.en.txt` for case notes and `testimony.en.html` for testimonials, then rebuild. `testimony-additions.en.txt` records the newly translated testimonial passages. Category listings include all matching published articles instead of relying on CMS pagination. The two hidden experimental template-25 pages use the standard site shell and their stored content; their additional experimental template-level widgets are not replicated.

## Preview locally

From this folder:

```sh
python3 -m http.server 8765 --bind 127.0.0.1 --directory public
```

Open http://127.0.0.1:8765/. Serve from a web root, rather than opening HTML files directly, because asset and navigation paths are root-relative. The Python preview server does not implement the Nginx redirects for `.php` URLs; `/?page=home` demonstrates the JavaScript compatibility redirect.

## Deploy to a DigitalOcean Ubuntu Droplet

Use a server with SSH access. A static site of this size requires very little storage or processing. No server has been created or changed as part of this conversion.

1. On the server, install Nginx and create the document root:

   ```sh
   sudo apt update
   sudo apt install nginx
   sudo mkdir -p /var/www/homeodoc
   ```

2. From this folder on your computer, upload the public files and configuration. Replace `USER` and `SERVER_IP` with your SSH details:

   ```sh
   scp -r public USER@SERVER_IP:~/homeodoc-public
   scp deploy/homeodoc.conf USER@SERVER_IP:~/homeodoc.conf
   ```

3. On the server, install the uploaded files and enable the configuration:

   ```sh
   sudo cp -R ~/homeodoc-public/. /var/www/homeodoc/
   sudo chmod -R a+rX /var/www/homeodoc
   sudo cp ~/homeodoc.conf /etc/nginx/sites-available/homeodoc
   sudo ln -s /etc/nginx/sites-available/homeodoc /etc/nginx/sites-enabled/homeodoc
   sudo nginx -t
   sudo systemctl reload nginx
   ```

   Run the reload only if `nginx -t` succeeds. This configuration has not been tested with a local Nginx binary; Nginx is not installed on the build machine. The config's `map` blocks must be included in the Nginx `http` context, as Ubuntu's default `sites-enabled` include does. If another server block already uses these domain names, replace that block instead of enabling a conflicting one.

4. Test before changing DNS:

   ```sh
   curl -I -H 'Host: homeodoc.ca' http://SERVER_IP/
   curl -I -H 'Host: homeodoc.ca' 'http://SERVER_IP/index.php?page=home'
   curl -I -H 'Host: homeodoc.ca' http://SERVER_IP/not-a-page
   ```

   Expect 200, 301 to `/home.html`, and 404 respectively. Permit inbound TCP ports 80 and 443 in any configured firewall. Point the domain's A records at the Droplet when ready; preserve existing mail/MX records. Do not publish the source archive or this entire project directory.

5. Enable HTTPS after the domain resolves to the server. Follow [DigitalOcean's Nginx and Let's Encrypt instructions](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-22-04) for Certbot installation, then obtain a certificate for `homeodoc.ca` and `www.homeodoc.ca`. Verify certificate renewal using the guide. The supplied configuration initially serves HTTP only.

Nginx setup reference: [DigitalOcean's installation guide](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04). Static file routing reference: [Nginx try_files documentation](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files).

## Rebuild and verify

Python 3.9 or later, standard library only:

```sh
python3 tools/build.py '/Users/emilearragon/Documents/Homeodoc archive'
python3 tools/validate.py
```

The builder parses only the selected public-content SQL tables and never executes old PHP or SQL. Rebuilding overwrites generated files, so keep manual content edits separately or update the builder first. `migration-report.json` records counts and routes. `tools/validate.py` checks local links, section anchors, CSS assets, leftover CMS tags, and allowed file types. The completed build passed 2,905 local-reference checks across 117 HTML documents with zero errors. French-to-English navigation and the contact page were also inspected in a browser. External email delivery, DNS, HTTPS, and live server redirects remain deployment checks.

## GoDaddy / Apache hosting

Upload the contents of `public/` to your domain document root (usually `public_html`), including the hidden `.htaccess` file. It sets index.html as the homepage, disables directory listings, uses UTF-8, and serves the custom 404 page. It contains no legacy redirects. Enable SSL and HTTPS through your hosting panel. The Python preview server ignores .htaccess; Apache behavior must be checked on hosting.

# Waffy Escrow Payment for WooCommerce — Installation & Setup Guide

**Plugin:** Waffy Escrow Payment for WooCommerce
**Plugin slug:** `waffy-woocommerce`
**Version:** 0.3.0
**License:** GPL-2.0-or-later

Compatible with **WordPress 6.0+** and **WooCommerce 8.0+**, on **PHP 8.1+**.
Supports both the **classic** and the **Blocks (Gutenberg)** checkout, and
**High-Performance Order Storage (HPOS)**.

> The Waffy PHP SDK and all PHP dependencies are **bundled inside the zip**
> (under `sdk/` and `vendor/`). There is nothing to install with Composer and
> nothing to fetch from the internet — a WordPress site never runs Composer.
> Uploading the zip installs everything the plugin needs.

---

## 1. Requirements

| Requirement | Value |
|-------------|-------|
| WordPress | 6.0 or higher |
| WooCommerce | 8.0 or higher (active) |
| PHP | 8.1 or higher |
| PHP extensions | `ext-json`, `ext-openssl` (standard on any WooCommerce host) |
| Permalinks | Any setting **other than "Plain"** — see §3.2 |
| Access | WordPress admin, with the `install_plugins` capability (Administrator) |
| Waffy credentials | Client ID, Client Secret, admin email & password — request from **support@waffyapp.com** |

Unlike Magento, **no command-line access is required**. The whole install is
done from the WordPress admin.

---

## 2. Installation from the zip

### 2.1 Get the zip

Download the latest `waffy-woocommerce-<version>.zip` from the releases page:

**<https://github.com/WaffyApp/waffy-woocommerce-dist/releases/latest>**

Take the `waffy-woocommerce-<version>.zip` asset. Ignore GitHub's automatic
**Source code (zip)** and **(tar.gz)** entries — those are archives of the
distribution repo, not the plugin, and WordPress will reject them.

### 2.2 Upload it (recommended)

1. In WP Admin, go to **Plugins → Add New → Upload Plugin**.
2. Click **Choose File**, select `waffy-woocommerce-0.3.0.zip`, then
   **Install Now**.
3. Click **Activate Plugin**.

That's it — the plugin creates its encrypted token-cache table
(`wp_waffy_tokens`) and schedules its background token refresh on activation.

> **"The uploaded file exceeds the upload_max_filesize directive"** — the zip is
> roughly 700 KB, well under any normal limit, but some hosts cap uploads very
> low. If you hit this, use the SFTP method below or ask your host to raise
> `upload_max_filesize`.

### 2.3 Or upload by SFTP/SSH

If the admin uploader is disabled or you prefer file access:

1. Unzip `waffy-woocommerce-0.3.0.zip` locally. It contains a single top-level
   folder, `waffy-woocommerce/`.
2. Upload that whole folder into `wp-content/plugins/` so you end up with
   `wp-content/plugins/waffy-woocommerce/waffy-woocommerce.php`.
3. In WP Admin, go to **Plugins**, find **Waffy Escrow Payment for
   WooCommerce**, and click **Activate**.

### 2.4 Verify it activated

Go to **Plugins** — the entry should show as active. If WooCommerce is missing
or too old, or PHP is below 8.1, the plugin stays inactive and prints an admin
notice explaining which requirement failed.

---

## 3. Configuration

Go to **WooCommerce → Settings → Payments → Waffy Escrow Payment**.

### 3.1 Connect your Waffy account

1. **Environment** — set to **Sandbox** first for testing (no real money).
   Sandbox and Production have separate credential fields, so you can fill both
   and switch between them without re-entering anything.
2. Enter the **Sandbox Client ID**, **Client Secret**, **Client Admin Email**,
   and **Client Admin Password** provided by Waffy.
3. **Merchant Phone Number** *(required)* — E.164 format, e.g. `+966555555555`.
   This identifies the merchant as the PROVIDER party on every escrow contract.
4. *(Optional)* **Broker Phone Number** — leave empty if no broker is involved.

> Secrets and passwords are stored **encrypted** (AES-256-GCM). Once saved, the
> fields show blank — leaving them blank on a later save keeps the current
> value rather than clearing it.

### 3.2 Register your webhook with Waffy

1. Copy the **Webhook URL** shown in the settings. It looks like:

   ```
   https://your-store.com/wp-json/waffy/v1/webhook
   ```

2. Send that URL to the Waffy team so they register your store for order status
   updates. **Order status will never update until this is done.**
3. *(Optional)* **Webhook Allowed IPs** — leave empty to allow all. To restrict,
   enter one IP or CIDR range per line (e.g. `203.0.113.0/24`); anything else
   gets a 403. Ask Waffy for their server IPs before turning this on.

> **Permalinks matter.** The webhook is a WordPress REST route. If
> **Settings → Permalinks** is set to **Plain**, the pretty `/wp-json/` URL does
> not work and the settings page will show the fallback
> `?rest_route=/waffy/v1/webhook` form instead. Either is fine as long as you
> send Waffy exactly the URL the settings page displays — but any non-Plain
> permalink structure is recommended.

### 3.3 Contract settings

Match these to how you sell:

| Setting | Meaning |
|---------|---------|
| **Return Policy** | `No Return` or `Returnable` |
| **Return Fee Payee** | Who pays the return fee — Merchant (Provider) or Customer |
| **Is Deliverable** | Product requires physical delivery |
| **Is Inspectable** | Buyer may inspect goods before funds are released |
| **Is Acceptable (by Customer)** | Buyer explicitly accepts/rejects before release |
| **Milestone Deadline (days)** | Days from order date until the payment milestone expires |
| **Contract Category** | Category shown to the buyer on the Waffy page (e.g. Services, Electronics, Food) |

### 3.4 Storefront settings

1. **Title** — what buyers see at checkout (default "Pay with Waffy").
2. **Description** — the explanatory line under the title at checkout.
3. Tick **Enable Waffy Escrow Payment**.
4. Click **Save changes**.

Leave everything under **Advanced** empty. Those URL overrides are derived
automatically from the Environment setting and exist only for pointing at a
custom or staging Waffy server.

---

## 4. Test in Sandbox

1. Place a test order on the storefront and choose **Pay with Waffy** at
   checkout.
2. You are redirected to the Waffy-hosted payment page. The order is created in
   WooCommerce as **On hold** at this point.
3. Complete the payment on the Waffy page.
4. Confirm the order moves to **Processing** once Waffy's webhook arrives, and
   that an order note records the Waffy status.

Test both the classic and the Blocks checkout if your store offers both.

---

## 5. Go live

1. Set **Environment → Production**.
2. Enter your **Production** Client ID, Client Secret, Admin Email & Password.
3. If your store URL changed (e.g. staging → live domain), re-share the new
   **Webhook URL** with Waffy — it is derived from the site URL.
4. **Save changes**.
5. Place one small real order to confirm end to end.

---

## 6. Updating to a new version

Settings and cached tokens survive an update; they live in WordPress options
and the `wp_waffy_tokens` table, not in the plugin folder.

**Via the admin uploader:** go to **Plugins → Add New → Upload Plugin**, choose
the newer zip, and click **Install Now**. WordPress detects the existing copy
and offers **Replace current with uploaded** — take it.

**Via SFTP:** deactivate the plugin, delete the
`wp-content/plugins/waffy-woocommerce/` folder, upload the new one, reactivate.
Do **not** merge the new folder over the old one — files removed in the new
version would linger and can break the plugin.

> **Do not use "Delete" in the Plugins screen to update.** Deleting runs the
> uninstall routine (§7), which wipes your credentials and token cache.

---

## 7. Uninstalling

- **Deactivate** stops the plugin and cancels its background token refresh.
  Nothing is deleted; reactivating restores the previous setup.
- **Delete** (after deactivating) additionally removes the `wp_waffy_tokens`
  table, the gateway settings, and the token-warmer bookkeeping.

Waffy identifiers already written to past orders (`_waffy_milestone_id`,
`_waffy_contract_id`) are **deliberately kept** as part of those orders'
historical record.

---

## 8. Troubleshooting

| Symptom | Fix |
|---------|-----|
| Plugin won't activate | Check the admin notice — usually WooCommerce inactive/too old, or PHP below 8.1 |
| "Waffy" not shown at checkout | Confirm **Enable Waffy Escrow Payment** is ticked and the settings were saved |
| Order stuck on **On hold** | The webhook never arrived. Confirm the Webhook URL was registered with Waffy and isn't blocked by the IP allowlist or a firewall/WAF |
| Webhook returns 403 | The caller's IP isn't in **Webhook Allowed IPs**. Clear the field to allow all, or add Waffy's IPs |
| Webhook returns 404 | Permalinks are set to **Plain**, or a security plugin is blocking `/wp-json/`. See §3.2 |
| Checkout is slow on the first order | Token warm-up hasn't run yet. Harmless — the SDK fetches what's missing. Confirm WP-Cron is working |
| Credentials seem not to save | Secret/password fields intentionally display blank after saving. Re-entering is only needed to change them |
| Admin notice: "stored ... could not be read" | The site's WordPress security keys/salts changed (common after a migration or a host "security reset"), so the encrypted secrets can't be decrypted. Re-enter and save the credentials |

### The call log

Every Waffy call is logged with its outcome and duration under
**WooCommerce → Status → Logs**, source `waffy` (on disk:
`wp-content/uploads/wc-logs/waffy-*.log`). Start here for anything unexplained —
it shows each checkout step, whether tokens were cached or fetched, and the
webhook decisions.

---

## Support

Email **support@waffyapp.com** for credentials, webhook registration, or setup
help.

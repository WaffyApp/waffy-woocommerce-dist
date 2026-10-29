# Waffy Escrow Payment for WooCommerce — Installation & Setup Guide

**Plugin:** Waffy Escrow Payment for WooCommerce
**Plugin slug:** `waffy-woocommerce`
**Version:** 0.7.2
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
| Permalinks | Any setting **other than "Plain"** — see §3.3 |
| Access | WordPress admin, with the `install_plugins` capability (Administrator) |
| Waffy client | An API client created in the Waffy portal, its env file, client secret and admin password |

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

Every release also carries a version-less `waffy-woocommerce.zip` with
identical contents, if you'd rather always fetch the current version from one
unchanging address:

```
https://github.com/WaffyApp/waffy-woocommerce-dist/releases/latest/download/waffy-woocommerce.zip
```

### 2.2 Upload it (recommended)

1. In WP Admin, go to **Plugins → Add New → Upload Plugin**.
2. Click **Choose File**, select the `waffy-woocommerce-<version>.zip` you downloaded, then
   **Install Now**.
3. Click **Activate Plugin**.

That's it — the plugin creates its encrypted token-cache table
(`wp_waffy_tokens`) and schedules its background token refresh on activation.

> **"The uploaded file exceeds the upload_max_filesize directive"** — the zip is
> roughly 500 KB, well under any normal limit, but some hosts cap uploads very
> low. If you hit this, use the SFTP method below or ask your host to raise
> `upload_max_filesize`.

### 2.3 Or upload by SFTP/SSH

If the admin uploader is disabled or you prefer file access:

1. Unzip `waffy-woocommerce-<version>.zip` locally. It contains a single top-level
   folder, `waffy-woocommerce/`.
2. Upload that whole folder into `wp-content/plugins/` so you end up with
   `wp-content/plugins/waffy-woocommerce/waffy-woocommerce.php`.
3. In WP Admin, go to **Plugins**, find **Waffy Escrow Payment for
   WooCommerce**, and click **Activate**.

### 2.4 Verify it activated

Go to **Plugins** — the entry should show as active.

If something is missing, what you see depends on which requirement it is:

- **PHP below 8.1, or WordPress below 6.0** — WordPress refuses to activate the
  plugin at all and shows its own error. This is enforced by WordPress from the
  plugin header, not by the plugin.
- **WooCommerce not installed or not active** — the plugin *does* activate, but
  it shows an admin notice and does not register the payment method. Install or
  activate WooCommerce and the gateway appears.
- **WooCommerce older than 8.0** — nothing blocks this. The plugin loads
  normally, and WooCommerce may flag the version mismatch on the Plugins
  screen. Treat 8.0 as the supported floor, not an enforced one.

---

## 3. Configuration

Go to **WooCommerce → Settings → Payments → Waffy Escrow Payment**. The top of
the screen is the **Waffy setup wizard**; any step can be reopened later.

### 3.1 Create your Waffy client

Choose **Sandbox** (testing, no real money) or **Live**. Then, in the Waffy
portal — `https://portal-dev.waffyapp.com` for Sandbox,
`https://portal.waffyapp.com` for Live — set up your organization if you have
not already, create an API client for WooCommerce, and download its **env
file**. Keep the client's **secret** at hand: the file does not include it.

### 3.2 Connect

Upload the env file, **or** type the values (client ID, admin username — a mobile or an email,
organization account ID). Either way, enter the **client secret** and the
**admin password**, then **Save**.

The banner shows which environment you are setting up. A Sandbox file (its
addresses contain `dev` or `stg`) is refused while Live is selected, and vice
versa — use the *Switch* link the wizard offers.

> Secrets are stored **encrypted** (AES-256-GCM). When you edit the connection
> later, leaving a secret blank keeps the saved one — for the same client only.

### 3.3 Verify & activate

Click **Run checks and activate**. The wizard checks the client and admin
credentials with Waffy, reads your client's settings, and **registers your
store's webhook URL with Waffy** — no need to send it to anyone. When every
check passes, the connection becomes active and the payment method is switched
on. Until then your previous connection (if any) keeps working.

> **Permalinks matter.** The webhook is a WordPress REST route; any non-Plain
> permalink structure is recommended (**Settings → Permalinks**).

### 3.4 Storefront settings

Below the wizard: **Title** (what buyers see at checkout) and **Description**.
Click **Save changes** after editing them.

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

1. Open the setup wizard, choose **Live** in step 1, and create a live client
   in `https://portal.waffyapp.com`.
2. Upload the live env file in step 2 and run the checks in step 3.
3. If your store URL changes later (e.g. staging → live domain), run step 3
   again — the webhook URL is derived from the site URL, and the checks
   register the new one with Waffy.
4. Place one small real order to confirm end to end.

---

## 6. Updating to a new version

Settings and cached tokens survive an update; they live in WordPress options
and the `wp_waffy_tokens` table, not in the plugin folder.

> **Upgrading from 0.3.x or earlier?** Version 0.4.0 connects through Waffy ID
> and needs the env file from the Waffy portal. The old credentials have no
> organization account ID, so they don't carry over as a working connection:
> Waffy stays hidden at checkout until you run the setup wizard (§3) once.

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
| "Waffy" not shown at checkout | Run the checks in the setup wizard (§3.3) — the method is switched on only when every check passes |
| Order stuck on **On hold** | The webhook never arrived. Run the checks in the setup wizard again — the last one registers the webhook URL with Waffy — and confirm a firewall/WAF isn't blocking it |
| Webhook returns 404 | Permalinks are set to **Plain**, or a security plugin is blocking `/wp-json/`. See §3.3 |
| Checkout is slow on the first order | Token warm-up hasn't run yet. Harmless — the SDK fetches what's missing. Confirm WP-Cron is working |
| A wizard check fails | The failing line says which credential Waffy refused, with Waffy's message. Fix it in step 2 and run the checks again |
| Admin notice: "stored ... could not be read" | The site's WordPress security keys/salts changed (common after a migration or a host "security reset"), so the encrypted secrets can't be decrypted. Run the setup wizard again |

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

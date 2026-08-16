# Waffy Escrow Payment for WooCommerce — downloads

Official download location for the **Waffy Escrow Payment** plugin for
WordPress / WooCommerce.

This repository contains **no source code** — only released plugin packages.

## Download

**→ [Latest release](https://github.com/WaffyApp/waffy-woocommerce-dist/releases/latest)**

Take the **`waffy-woocommerce-<version>.zip`** asset.

> Ignore the **Source code (zip)** and **Source code (tar.gz)** entries GitHub
> adds automatically. They are archives of *this* repository, not the plugin,
> and WordPress will reject them.

## Install

1. In WP Admin, go to **Plugins → Add New → Upload Plugin**.
2. Choose `waffy-woocommerce-<version>.zip` and click **Install Now**.
3. Click **Activate Plugin**.
4. Configure under **WooCommerce → Settings → Payments → Waffy Escrow Payment**.

Full instructions, including webhook registration and going live, are in
[INSTALL.md](INSTALL.md) — also included inside the zip.

## Requirements

| | |
|---|---|
| WordPress | 6.0+ |
| WooCommerce | 8.0+ |
| PHP | 8.1+ with `ext-json`, `ext-openssl` |

Works with the classic and Blocks checkout, and with High-Performance Order
Storage (HPOS).

## Updating

Upload the newer zip the same way — WordPress offers **Replace current with
uploaded**. Your settings are preserved.

Do **not** use **Delete** to update: deleting runs the uninstall routine and
removes your saved credentials.

## Support

Merchant credentials, webhook registration and setup help:
**support@waffyapp.com**

## License

GPL-2.0-or-later.

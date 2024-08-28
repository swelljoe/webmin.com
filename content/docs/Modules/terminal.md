---
title: "Terminal"
date: 2024-07-23
author: "Ilia Ross"
weight: 615
---

### About

The Terminal module in Webmin is a feature that allows you to access and interact with the command-line shell of your server or system directly from within the Webmin interface.

{{< alert primary exclamation "Note" "Starting with Webmin 2.200, all _sudo_-capable users will log in as themselves instead of as _root_. To disable this limitation, go to **Webmin ⇾ Webmin Users ⇾ root: Edit Webmin User / Available Webmin Modules: Tools ⇾ Terminal: Module Access Control** page, and set the **Enforce _sudo_-only privileges** option to **No**." >}}

With the Terminal module, you can perform various tasks using commands just like you would in a traditional terminal or command prompt. This includes running commands, executing scripts, managing files and directories, configuring system settings, and much more.

[![](/images/docs/screenshots/modules/light/terminal.png "Terminal Screenshot")](/images/docs/screenshots/modules/light/terminal.png)

Overall, the Terminal module in Webmin provides a convenient and user-friendly way to access and manage your server via the command line, directly from your web browser. It allows you to perform administrative tasks quickly and efficiently without needing to rely solely on SSH or physical access to the server.

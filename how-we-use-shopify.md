# How we use Shopify

Here at Kvalifik, we build and support a lot of different webshops, all powered by Shopify.

This document covers topics specific to our workflow, and our way of developing for Shopify.

For general Shopify information, refer to Shopify's official documentation at [Shopify.dev](https://shopify.dev/themes).

Shopify Themes are powered by basic HTML, CSS and JS, supplemented with the templating language [Liquid](https://shopify.dev/api/liquid).

## Themes

Shopify uses 'theme' for describing the Liquid, HTML, CSS and JS that make up the storefront. This is akin to Wordpress' usage of the term.

For an overview of Shopify's theme architecture refer to their docs found [here](https://shopify.dev/themes/architecture#directory-structure-and-component-types).

As of April 2022, our Shopify stack is split between three types of themes:

- Kvalifik Skeleton Theme
- Dawn / OS 2.0 Themes
- Legacy Themes

Kvalifik Skeleton Theme is our opinionated version of Debut which we've optimized and replaced all styling with TailwindCSS. It is used in larger, 'shop from scratch' projects such as Mikkeller Webshop 2.0, Rebiome and FDB Møbler. We're not going to start up new stores with this theme, as Dawn (the latest flagship from Shopify) is a solid starting point.

Dawn is Shopify's latest and greatest theme, showcasing Online Store 2.0 and modern 'Evergreen Web' principles. We have multiple clients looking to this theme as their foundation for the next webshop upgrade. Stores built on this theme are in-line with Shopify's vision for future themes, and can be periodically updated with features from the Dawn repo (github.com/Shopify/dawn).

Legacy Themes cover the majority of themes currently in use by clients. These are usually based on older Shopify theme standards such as Timber, Slate or Debut/Brooklyn. They are to be seen as outdated, and deprecated as of 2021. We should strive to upgrade these stores to a OS2.0 compatible theme ASAP, instead of building upon these themes.

## GitHub

Our Shopify projects should at least have two branches

- Production
- Development / Awaiting Approval

These branches should be linked with a theme on the client store. Production is _usually_ the live theme, but some clients prefer duplicating it, and using temporary live themes instead. For those clients, refer to the workflow described [here](#duplicate-theme-workflow).

When working on features, use Kvalifik CLI to create a branch pr. feature, and preview these via `kvalifik shopify-serve`. When ready for approval by client, merge your branch into `/development`.

When a feature is approved, merge `/development` into `/production`.

Which branch to merge depends on the status of the other features present in `/development`. One could imagine a scenario where we've built 4 features, merged all into `/development` for client approval. Here we could risk that only 3 of them are approved. In this case, you should revert the pull request (into `/development`) and restore the branch. Then you can merge the other features into `/production`, and fix the rejected.

## Shopify x Kvalifik CLI

> You can read more about Kvalifik-CLI in the repo found at https://github.com/kvalifik/kvalifik-cli.

Our CLI has a few bespoke Shopify commands that ease the use of Shopify CLI for theme development.

The biggest current issue with Shopify CLI vs the older ThemeKit, is that it's project agnostic. This means that running `shopify theme serve` in `/kvalifik/rebiome` does not guarrantee that your theme is served on https://rebiome.net.

Instead, it is served to whichever store you were logged in to last. So if you're switching between projects, it's easy to forget to switch store.

Our Shopify commands all include a check, which makes sure your working directory matches the store Shopify CLI is logged in to, preventing this.

These commands are:

- `kvalifik shopify-init`
- `kvalifik shopify-serve`
- `kvalifik shopify-get-live-settings`

### shopify-init

Set's up your project, and prompts you to fill out our `shopify.config.json` file. This is required for the other commands to work.

### shopify-serve

A project agnostic way of serving a development theme.

Our Skeleton Theme projects are built with Gulp and Tailwind, which requires a slightly different dev script. OS 2.0 and legacy themes can simply use Shopify CLI. This command checks your project and picks the right command.

### shopify-get-live-settings

A command that fetches theme settings from the currently published 'live' theme.

This makes sure that we are always up to date with the latest Theme Customizer changes made by a client. With this, we can be confident that our changes wont break in production and that the clients changes aren't overwritten by old data from our `/staging` or `/development` branches.

This feature was built specifically for clients who don't use our `/production` theme as their live theme. This workflow is described [here](#duplicate-theme-workflow).

<h2 id="duplicate-theme-workflow">Workflow for duplicating `/production`themes</h2>
<blockquote>
Note: This workflow is temporary, and will be replaced by CI in the future.
</blockquote>

Some of our clients prefer to use our `/production` theme as a template, which they duplicate and customize to their liking.

This means we can't push new features to live directly, but this is usually a non-issue.

More importantly, it means that our GitHub will be out of sync with the latest Theme Customizations. To alleviate this, we're working on a CI solution but that's currently waiting for an update to Shopify-CLI. Until then, you can and should use our custom CLI command `kvalifik shopify-get-live-settings` to pull the `/config/settings_data.json` file into your branch, before merging with `/development` and `/production`.

This way, the next time our client duplicates our `/production` theme, they'll have all the edits they made last time.

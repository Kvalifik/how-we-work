# How we use Shopify

## GitHub

Our Shopify projects should at least have two branches

- Production
- Development / Awaiting Approval

These branches should be linked with a theme on the client store. Production is _usually_ the live theme, but some clients prefer duplicating it, and using temporary live themes instead.

When working on features, use Kvalifik CLI to create a branch pr. feature, and preview these via `kvalifik shopify-serve`. When ready for approval by client, merge your branch into `Development`. Do _not_ delete your branch!

When a feature is approved, merge its branch or `Development`into `Production`. Which branch to merge depends on the status of the other features present in `Development`. One could imagine a scenario where we've built 4 features, merged all into `Development` for client approval. Here we could risk that only 3 of them are approved. You have two options here:

- You can revert the commit for the rejected feature
- Keep development as is, and instead merge the 3 approved features one by one.

# copy run start

The source for [karsagar28.github.io](https://karsagar28.github.io): saving network insights before the next reload.

Drafts and editorial reviews should follow the [copy run start style guide](STYLE_GUIDE.md).

## Add a post

1. Create `_posts/YYYY-MM-DD-short-title.md`.
2. Add front matter for `title`, `date`, `category`, `tags`, and `reading_time`.
3. Make sure the title is descriptive enough to stand alone in the homepage list.
4. Write the article in Markdown and push it to `main`. GitHub Pages publishes it automatically.

Once a draft becomes a post, edit the file under `_posts/` directly. GitHub Pages does not read standalone drafts stored outside this repository, so keeping the published post as the source of truth prevents later edits from diverging.

## Add images

Keep each post's images together:

```text
assets/images/posts/<post-slug>/
```

Use lowercase, descriptive filenames. Prefer WebP or JPEG for photographs and PNG for screenshots. Create explanatory diagrams in Excalidraw, keep the editable `.excalidraw` source beside the article asset, and publish an SVG export. Resize large raster originals to a sensible display width—usually 1,600 pixels or less—and aim for files below roughly 500 KB when quality permits.

Basic Markdown image:

```markdown
![A concise description of the diagram]({{ '/assets/images/posts/post-slug/diagram.png' | relative_url }})
```

Image with a caption:

```html
<figure class="post-figure">
  <img src="{{ '/assets/images/posts/post-slug/diagram.png' | relative_url }}"
       alt="A concise description of the diagram">
  <figcaption>What the reader should notice in this diagram.</figcaption>
</figure>
```

Always include useful alt text. Images are versioned with the article and served directly by GitHub Pages; no external image service is required at this stage.

## Analytics

In [Cloudflare Web Analytics](https://dash.cloudflare.com/?to=/:account/web-analytics), add `copyrunstart.karthiksagar.com`, open **Manage site**, and copy the public token from the JavaScript snippet into `cloudflare_web_analytics.token` in `_config.yml`. This is the site's public beacon token, not a Cloudflare API token. If the hostname is proxied through Cloudflare, select **Enable with JS Snippet installation** to avoid also injecting the beacon automatically.

The shared layout loads the beacon on all pages only in production and only when a token is configured. GitHub Pages uses the production environment. Leave the token empty to disable analytics; local development builds never send analytics. After publishing, visit a page and check Cloudflare after a few minutes. See [Cloudflare's setup instructions](https://developers.cloudflare.com/web-analytics/get-started/).

## Comments and reactions

Install the [Giscus GitHub app](https://github.com/apps/giscus) with access to **only `karsagar28.github.io`**. GitHub Discussions must remain enabled. The `giscus` settings in `_config.yml` identify this repository and its Announcements category.

Each post includes Giscus comments with main-post reactions enabled, a comment box above the conversation, lazy loading, and a light theme matching the blog. Readers sign in with GitHub to participate. The first comment or reaction creates the discussion automatically; moderate it in the repository's Discussions tab.

Discussions use strict `pathname` matching, so keep published post paths stable. Changing an article title or domain does not break its discussion if its path stays the same. Set `comments: false` in a post's front matter to hide both comments and reactions, or set `giscus.enabled: false` to disable them across the blog. The homepage and About page have no comment widget. See [Giscus configuration](https://giscus.app/).

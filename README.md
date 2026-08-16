# Network Architect Scratchpad

The source for [karsagar28.github.io](https://karsagar28.github.io), a question-led technical blog built with Jekyll and GitHub Pages.

## Add a post

1. Create `_posts/YYYY-MM-DD-short-title.md`.
2. Add front matter for `title`, `description`, `date`, `category`, `tags`, and `reading_time`.
3. Write the article in Markdown and push it to `main`. GitHub Pages publishes it automatically.

## Add images

Keep each post's images together:

```text
assets/images/posts/<post-slug>/
```

Use lowercase, descriptive filenames. Prefer WebP or JPEG for photographs and PNG for diagrams or screenshots. Resize large originals to a sensible display width—usually 1,600 pixels or less—and aim for files below roughly 500 KB when quality permits.

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

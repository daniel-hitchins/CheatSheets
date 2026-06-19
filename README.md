# Developer Cheat Sheets

A collection of practical, developer-focused cheat sheets for quick reference.

## 📚 Available Cheat Sheets

- [Accessibility Developer Cheat Sheet](identity-accessibility-developer-cheat-sheet.md) - PlatformUI and Mosaic Design System accessibility guide

## 🚀 Usage

### View Online
Visit the GitHub Pages site: [Your GitHub Pages URL]

### Add a New Cheat Sheet

1. Create a new `.md` file in this directory
2. Add an entry to the `cheatSheets` array in `index.html`:

```javascript
{
    title: "Your Cheat Sheet Title",
    file: "your-cheat-sheet-file.md",
    description: "Brief description of what this covers",
    tags: ["Tag1", "Tag2", "Tag3"]
}
```

That's it! The index page will automatically link to your new cheat sheet.

## 🛠️ Local Development

To preview locally with GitHub Pages behavior:
```bash
# Install Jekyll
gem install bundler jekyll

# Serve locally
jekyll serve
```

Or simply open `index.html` in a browser to see the index page.

## 📄 License

Feel free to use, share, and adapt these cheat sheets.

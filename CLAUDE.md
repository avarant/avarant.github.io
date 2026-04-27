# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start development server at localhost:4321
- `npm run build` - Build production site to ./dist/
- `npm run preview` - Preview build locally
- `npm run astro check` - Run Astro's TypeScript checker

## Architecture Overview

This is an Astro-based personal blog with the following key structure:

### Content Management
- **Blog posts**: Stored in `src/content/blog/` as Markdown files
- **Projects**: Defined in `src/content/projects/` (currently unused but configured)
- **Content schema**: Defined in `src/content.config.ts` using Zod validation
  - Blog posts require: title, description, publicationDate
  - Optional: image, imageAlt, tags

### Site Configuration
- **Core config**: `src/siteConfig.ts` exports SITE, NAV_LINKS, and SOCIAL_LINKS
- **Astro config**: Uses sitemap integration, TailwindCSS, and deploys to GitHub Pages
- **Site URL**: https://avarant.github.io

### Component Architecture
- **BaseLayout.astro**: Main layout wrapper with navigation and footer
- **Page components**: Introduction, Posts, Projects for homepage sections
- **SEO components**: SeoPage and SeoPost for metadata
- **Utility components**: ThemeToggle, ExternalLink, SocialLinks

### Styling
- Uses TailwindCSS v4 with custom color variables
- Dark mode support via CSS custom properties
- Typography styles in separate `typography.css`

### Dynamic Routes
- `/blog/[...id].astro` - Individual blog post pages
- `/blog/index.astro` - Blog listing page
- RSS feed generation at `/rss.xml`

When adding new blog posts, create markdown files in `src/content/blog/` with proper frontmatter matching the schema in `content.config.ts`.
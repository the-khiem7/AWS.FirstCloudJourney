# AWS First Cloud Journey

Personal internship documentation and learning journal for AWS Cloud technologies.

## ≡ƒÄ» About

This repository contains my internship documentation, workshop notes, and hands-on labs completed during my journey with AWS services at Amazon Web Services Vietnam Co., Ltd.

**Intern**: Nguyen Van Duy Khiem  
**Duration**: September 8, 2025 - December 12, 2025  
**Position**: FCJ Cloud Intern  
**University**: FPT University - Software Engineering

## Content Structure

### 1. [Worklog](content/1-Worklog/)
Weekly documentation of internship activities, learnings, and progress (Week 1-12).

### 2. [Proposal](content/2-Proposal/)
Internship and project proposals, approvals, and planning notes.

### 3. [Translated Blogs](content/3-Translated-Blogs/)
Translated AWS blog posts and reference write-ups for study.

### 4. [Events Participated](content/4-Events-Participated/)
Records of events, conferences, and trainings attended.
- [AWS Vietnam Cloud Day 2025](content/4-Events-Participated/4.1-AWS-Vietnam-Cloud-Day-2025/)

### 5. [Workshop: Private Analytics Platform](content/5-Workshop/)
Hands-on workshop building a private analytics platform with AWS services:
- Objectives & Scope
- Architecture Walkthrough
- Clickstream Ingestion Implementation
- Private Analytics Layer
- Shiny Dashboards Visualization
- Clean-up & Summary

### 6. [Self Assessment](content/6-Self-Assessment/)
Personal reflections on progress, strengths, gaps, and next steps.

### 7. [Sharing & Feedback](content/7-Sharing-&-Feedback/)
Shared learnings, peer feedback, and community notes.

## ≡ƒÜÇ Quick Start

### Prerequisites
- [Hugo Extended](https://gohugo.io/installation/) v0.152.2 or later
- Node.js (for package management)
- Git

### Local Development

```bash
# Clone repository
git clone https://github.com/the-khiem7/AWS.FirstCloudJourney.git
cd AWS.FirstCloudJourney

# Install dependencies
npm install

# Run development server with drafts
npm run dev

# Build for production
npm run build
```

Visit `http://localhost:1313/AWS.FirstCloudJourney/` to view the site locally.

## ≡ƒ¢á∩╕Å Tech Stack

- **Static Site Generator**: Hugo (Extended)
- **Theme**: hugo-theme-learn
- **Deployment**: 
  - GitHub Pages (Primary)
  - Vercel (Backup)
- **CI/CD**: GitHub Actions

## ≡ƒô¥ Content Management

### Adding New Content

```bash
# Create new page
hugo new content/<section>/<page>/_index.md
hugo new content/<section>/<page>/_index.vi.md
```

### Bilingual Support
All content is available in both English and Vietnamese:
- `_index.md` - English version
- `_index.vi.md` - Vietnamese version

### When Deleting Content Folders

**ΓÜá∩╕Å Important**: To avoid 404 errors on GitHub Pages:

1. **Remove references first** - Update all `_index.md` files that link to the content
2. **Delete the folder** - Remove from `content/` directory
3. **Test locally** - Run `npm run build` to verify
4. **Commit & push** - Deploy changes

## ≡ƒîÉ Deployment

### GitHub Pages (Primary)
- **URL**: https://the-khiem7.github.io/AWS.FirstCloudJourney/
- **Trigger**: Automatic on push to `main` branch
- **Workflow**: `.github/workflows/hugo.yml`

### Vercel (Backup)
- **Config**: Uses `config.production.toml` for relative URLs
- **Build**: `hugo --config config.production.toml --gc --minify`

## ≡ƒôé Repository Structure

```
.
|-- content/                # Markdown content files
|   |-- 1-Worklog/          # Weekly internship logs
|   |-- 2-Proposal/
|   |-- 3-Translated-Blogs/
|   |-- 4-Events-Participated/
|   |   \-- 4.1-AWS Vietnam Cloud Day 2025/
|   |-- 5-Workshop/         # Private Analytics workshop
|   |-- 6-Self-Assessment/
|   \-- 7-Sharing-&-Feedback/
|-- static/                 # Static assets (images, CSS, fonts)
|-- themes/                 # Hugo theme
|-- layouts/                # Custom layout overrides
|-- config.toml             # Hugo config for GitHub Pages
|-- config.production.toml  # Hugo config for Vercel
\-- package.json            # Node.js dependencies
```
```

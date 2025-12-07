# AWS First Cloud Journey

Personal internship documentation and learning journal for AWS Cloud technologies.

## 🎯 About

This repository contains my internship documentation, workshop notes, and hands-on labs completed during my journey with AWS services at Amazon Web Services Vietnam Co., Ltd.

**Intern**: Nguyen Van Duy Khiem  
**Duration**: September 8, 2025 - December 12, 2025  
**Position**: FCJ Cloud Intern  
**University**: FPT University - Software Engineering

## 📚 Content Structure

### 1. [Worklog](content/1-Worklog/)
Weekly documentation of internship activities, learnings, and progress (Week 1-12)

### 2. [Workshop: Private Analytics Platform](content/5-Workshop/)
Hands-on workshop building a private analytics platform with AWS services:
- Objectives & Scope
- Architecture Walkthrough
- Clickstream Ingestion Implementation
- Private Analytics Layer
- Shiny Dashboards Visualization
- Clean-up & Summary

### 3. [AWS Vietnam Cloud Day 2025](content/7-AWS%20Vietnam%20Cloud%20Day%202025/)
Conference notes and learnings from AWS Vietnam Cloud Day 2025:
- AWS Keynote Address (Eric Yeo)
- Opening Session (Jun Kai Loke)
- Amazon Bedrock & AgentCore
- Unified Data Foundation for AI & Analytics
- Amazon SageMaker Unified Studio & Lakehouse

## 🚀 Quick Start

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

## 🛠️ Tech Stack

- **Static Site Generator**: Hugo (Extended)
- **Theme**: hugo-theme-learn
- **Deployment**: 
  - GitHub Pages (Primary)
  - Vercel (Backup)
- **CI/CD**: GitHub Actions

## 📝 Content Management

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

**⚠️ Important**: To avoid 404 errors on GitHub Pages:

1. **Remove references first** - Update all `_index.md` files that link to the content
2. **Delete the folder** - Remove from `content/` directory
3. **Test locally** - Run `npm run build` to verify
4. **Commit & push** - Deploy changes

## 🌐 Deployment

### GitHub Pages (Primary)
- **URL**: https://the-khiem7.github.io/AWS.FirstCloudJourney/
- **Trigger**: Automatic on push to `main` branch
- **Workflow**: `.github/workflows/hugo.yml`

### Vercel (Backup)
- **Config**: Uses `config.production.toml` for relative URLs
- **Build**: `hugo --config config.production.toml --gc --minify`

## 📂 Repository Structure

```
.
├── content/              # Markdown content files
│   ├── 1-Worklog/       # Weekly internship logs
│   ├── 5-Workshop/      # Private Analytics workshop
│   └── 7-AWS Vietnam Cloud Day 2025/
├── static/              # Static assets (images, CSS, fonts)
├── themes/              # Hugo theme
├── layouts/             # Custom layout overrides
├── config.toml          # Hugo config for GitHub Pages
├── config.production.toml # Hugo config for Vercel
└── package.json         # Node.js dependencies

```

## 🔧 Configuration

### GitHub Pages Settings
- Source: GitHub Actions
- Branch: `main`
- Base URL: `https://the-khiem7.github.io/AWS.FirstCloudJourney`

### Build Commands
```json
{
  "build": "hugo --gc --minify",
  "dev": "hugo server -D"
}
```

## 📖 Guidelines

See [AGENTS.md](AGENTS.md) for detailed repository guidelines including:
- Project structure & module organization
- Build, test, and development commands
- Coding style & naming conventions
- Commit & pull request guidelines

## 📧 Contact

- **Email**: khiemnguyen120216@gmail.com
- **Facebook**: [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/)

## 📄 License

This project is for educational purposes as part of internship documentation.

---

Built with ❤️ using [Hugo](https://gohugo.io/) and deployed on [GitHub Pages](https://pages.github.com/)

# gh-arch-viz

**GitHub Architecture Visualizer** - Automatically detect and visualize your team's repository technology stacks.

## 📋 Overview

`gh-arch-viz` scans your GitHub organization's repositories and automatically detects:

- **Frameworks** (Next.js, React, etc.)
- **Build Tools** (Vite, Webpack, Turbopack, etc.)
- **Package Managers** (pnpm, npm, yarn)
- **CI/CD** (GitHub Actions, etc.)
- **Deployment Targets** (Vercel, etc.)
- **Container Technologies** (Docker, Docker Compose)
- **Infrastructure as Code** (Terraform, etc.)
- **Databases, Testing Frameworks, Linters**, and more

## ✨ Features

- **🔒 Org/Team Restricted Access** - Only members of your configured GitHub org and team can sign in
- **🤖 Automatic Detection** - Scans repository files to detect technologies with evidence-based scoring
- **🔍 Searchable Inventory** - Browse and filter all repositories by technology stack
- **📝 Evidence Tracking** - View exact files and snippets used for detection
- **📊 Visual Insights** - Charts and graphs showing technology distribution
- **⚡ Fast Scanning** - Concurrent repository scanning with rate limit handling
- **🎨 Modern UI** - Built with shadcn/ui and Tailwind CSS v4

## 🛠️ Tech Stack

- **Frontend**: Next.js 15 (App Router), React 19, TypeScript, shadcn/ui, Tailwind CSS v4
- **Backend**: Next.js Server Actions + API Routes
- **Database**: PostgreSQL (Neon) + Drizzle ORM
- **Auth**: Better Auth (GitHub OAuth)
- **Visualization**: Recharts
- **Package Manager**: pnpm

## 📦 Prerequisites

- Node.js 20+
- pnpm 9+
- PostgreSQL database (recommend [Neon](https://neon.tech))
- GitHub OAuth App credentials

## 🔧 Setup

### 1. Clone and Install

```bash
git clone <your-repo-url>
cd gh-arch-viz
pnpm install
```

### 2. Create GitHub OAuth App

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Click **New OAuth App**
3. Fill in the details:
   - **Application name**: gh-arch-viz
   - **Homepage URL**: `http://localhost:3000`
   - **Authorization callback URL**: `http://localhost:3000/api/auth/callback/github`
4. Click **Register application**
5. Copy the **Client ID** and generate a **Client Secret**

### 3. Set up Neon PostgreSQL

1. Create a free account at [Neon](https://neon.tech)
2. Create a new project
3. Copy the connection string (starts with `postgresql://`)

### 4. Configure environment variables

Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Edit `.env` and fill in:

```bash
# Database
DATABASE_URL="postgresql://user:pass@host.neon.tech/db?sslmode=require"

# Better Auth
BETTER_AUTH_SECRET="<generate-a-long-random-string>"
BETTER_AUTH_URL="http://localhost:3000"

# GitHub OAuth
GITHUB_CLIENT_ID="<your-github-oauth-client-id>"
GITHUB_CLIENT_SECRET="<your-github-oauth-client-secret>"

# GitHub Organization & Team
ALLOWED_GH_ORG="<your-org-name>"
ALLOWED_GH_TEAM_SLUG="<your-team-slug>"
```

**Note**: To find your team slug, go to your GitHub org → Teams → click on your team. The URL will be `github.com/orgs/YOUR_ORG/teams/YOUR_TEAM_SLUG`.

### 5. Run database migrations

```bash
pnpm db:push
```

### 6. Start the development server

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000)

## 📖 Usage

### First Time Setup

1. **Sign in with GitHub**

   - Click "Sign in with GitHub" on the landing page
   - Authorize the app with `read:org` and `repo` scopes
   - Only members of your configured org/team will be allowed access

2. **Scan Repositories**

   - Click "Scan All Repositories" on the inventory page
   - The app will list all repositories accessible to your team and scan them

3. **Browse Inventory**

   - View the table of all scanned repositories
   - Click on a repository name to see detailed detection results
   - View "Evidence" to see which files were used for detection

4. **View Insights**
   - Click "View Insights" to see charts and graphs
   - Analyze technology distribution across your organization

### Rescanning

- **Single Repository**: Navigate to repo detail page and click "Rescan"
- **All Repositories**: On the inventory page, click "Scan All Repositories"

## 🔒 Security & Permissions

### Required GitHub Scopes

- `read:org` - To verify organization and team membership
- `repo` - To read repository files for technology detection

### Access Control

- Only users who are active members of both:
  1. The configured GitHub organization (`ALLOWED_GH_ORG`)
  2. The configured team within that org (`ALLOWED_GH_TEAM_SLUG`)
- Membership is verified on each sign-in
- Session TTL can be configured via `SESSION_TTL_MINUTES`

### Recommendations

- **Production**: Consider implementing a GitHub App for better security
- **Tokens**: Access tokens are session-based
- **Rate Limits**: App respects GitHub API rate limits (5000 req/hour)

## 🚀 Development

### Available Scripts

```bash
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm start        # Start production server
pnpm lint         # Run ESLint
pnpm db:generate  # Generate Drizzle migrations
pnpm db:push      # Push schema changes to database
pnpm db:studio    # Open Drizzle Studio
```

## 🚢 Deployment

### Vercel (Recommended)

1. Push your code to GitHub
2. Import the project in [Vercel](https://vercel.com)
3. Add all environment variables from `.env`
4. Update `BETTER_AUTH_URL` and GitHub OAuth callback URL to your production domain
5. Deploy
6. Run `pnpm db:push` to sync database schema

### Environment Variables for Production

Make sure to set these in your hosting platform:

- `DATABASE_URL`
- `BETTER_AUTH_SECRET` (use a strong random string)
- `BETTER_AUTH_URL` (your production URL)
- `GITHUB_CLIENT_ID`
- `GITHUB_CLIENT_SECRET`
- `ALLOWED_GH_ORG`
- `ALLOWED_GH_TEAM_SLUG`

## 🐛 Troubleshooting

### "Not authorized: must be a member of the organization"

- Verify that your GitHub user is a member of the configured org and team
- Check that `ALLOWED_GH_ORG` and `ALLOWED_GH_TEAM_SLUG` are correct
- Ensure the team is not set to "Secret" visibility (must be "Visible")

### GitHub API Rate Limit

- Authenticated requests have a limit of 5000/hour
- The app includes retry logic with exponential backoff
- For large orgs, consider implementing background job queues

## 🗺 Roadmap

- [ ] **Policy Engine**: Define and enforce technology standards
- [ ] **Scheduled Scans**: Automatic daily/weekly rescans
- [ ] **GitHub App**: Replace OAuth with GitHub App
- [ ] **Notifications**: Alert on policy violations
- [ ] **Export**: CSV/JSON export of inventory
- [ ] **More Detectors**: Python, Go, Rust, Kubernetes, AWS, GCP
- [ ] **Custom Rules**: User-defined detection patterns
- [ ] **Historical Tracking**: Track technology changes over time

## 📝 License

MIT

## 🙏 Credits

Built with:

- [Next.js](https://nextjs.org)
- [Better Auth](https://www.better-auth.com)
- [Drizzle ORM](https://orm.drizzle.team)
- [shadcn/ui](https://ui.shadcn.com)
- [Recharts](https://recharts.org)
- [Octokit](https://github.com/octokit/octokit.js)

---

**Questions or issues?** Please open an issue on GitHub.

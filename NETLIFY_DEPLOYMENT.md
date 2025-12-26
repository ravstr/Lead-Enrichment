# Netlify Deployment Guide

This guide will walk you through deploying the fire-enrich application to Netlify.

## Prerequisites

- A Netlify account ([sign up here](https://app.netlify.com/signup))
- A GitHub/GitLab/Bitbucket repository (or use Netlify CLI)
- Your Firecrawl API key ([get it here](https://www.firecrawl.dev/app/api-keys))
- Your OpenAI API key ([get it here](https://platform.openai.com/api-keys))

## Deployment Methods

### Method 1: Git Integration (Recommended)

This method automatically deploys your site whenever you push to your repository.

#### Step 1: Push to Git Repository

Ensure your code is in a Git repository (GitHub, GitLab, or Bitbucket).

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-repository-url>
git push -u origin main
```

#### Step 2: Connect to Netlify

1. Log in to [Netlify Dashboard](https://app.netlify.com)
2. Click **"Add new site"** → **"Import an existing project"**
3. Choose your Git provider (GitHub, GitLab, or Bitbucket)
4. Authorize Netlify to access your repositories
5. Select the fire-enrich repository

#### Step 3: Configure Build Settings

Netlify should auto-detect Next.js and pre-fill these settings:
- **Build command:** `npm run build`
- **Publish directory:** `.next` (handled automatically by the Next.js plugin)
- **Node version:** 18 (configured in `netlify.toml`)

If not auto-detected, manually enter:
- Build command: `npm run build`
- Publish directory: `.next`

#### Step 4: Set Environment Variables

1. Before deploying, go to **Site settings** → **Environment variables**
2. Click **"Add variable"** and add:

   | Variable Name | Value | Description |
   |--------------|-------|-------------|
   | `FIRECRAWL_API_KEY` | `fc-...` | Your Firecrawl API key |
   | `OPENAI_API_KEY` | `sk-...` | Your OpenAI API key |
   | `FIRE_ENRICH_UNLIMITED` | `true` | (Optional) Enable unlimited mode in production |

3. Click **"Save"**

**Note:** Environment variables are encrypted and only available to your site's build and runtime.

#### Step 5: Deploy

1. Click **"Deploy site"**
2. Wait for the build to complete (typically 3-5 minutes for first build)
3. Once deployed, your site will be available at `https://your-site-name.netlify.app`

#### Step 6: Verify Deployment

Test the following:
- ✅ Homepage loads: `https://your-site-name.netlify.app`
- ✅ Enrichment page: `https://your-site-name.netlify.app/fire-enrich`
- ✅ API routes work (test CSV upload)
- ✅ Environment variables are accessible

### Method 2: Netlify CLI

Use this method for manual deployments or testing.

#### Step 1: Install Netlify CLI

```bash
npm install -g netlify-cli
```

#### Step 2: Login to Netlify

```bash
netlify login
```

This will open your browser to authorize the CLI.

#### Step 3: Initialize Site

```bash
netlify init
```

Follow the prompts:
- Create a new site or link to existing site
- Set build command: `npm run build`
- Set publish directory: `.next`

#### Step 4: Set Environment Variables

Set environment variables locally (or add them in the Netlify dashboard):

```bash
netlify env:set FIRECRAWL_API_KEY "fc-your-key-here"
netlify env:set OPENAI_API_KEY "sk-your-key-here"
netlify env:set FIRE_ENRICH_UNLIMITED "true"
```

#### Step 5: Deploy

```bash
# Deploy to production
netlify deploy --prod

# Or deploy a draft/preview first
netlify deploy
```

#### Step 6: Test Locally with Netlify

You can test locally with Netlify's environment:

```bash
netlify dev
```

This runs your site locally with Netlify's serverless functions simulation.

### Method 3: Drag & Drop (Quick Test)

For quick testing without Git:

1. Build your site locally:
   ```bash
   npm run build
   ```

2. Create a zip file of the `.next` directory

3. Go to [Netlify Drop](https://app.netlify.com/drop)

4. Drag and drop your zip file

**Note:** This method doesn't support automatic deployments or environment variables easily. Use Git integration for production.

## Configuration Files

### netlify.toml

The `netlify.toml` file in the root directory configures:
- Build settings (command, Node version)
- Next.js plugin (@netlify/plugin-nextjs)
- Function timeouts (26 seconds for Pro tier)
- Security headers

### Build Process

Netlify will:
1. Install dependencies (`npm install`)
2. Run build command (`npm run build`)
3. Detect Next.js and apply optimizations
4. Convert API routes to serverless functions
5. Deploy static assets and functions

## Environment Variables

### Required Variables

| Variable | Description | Where to Get It |
|----------|-------------|-----------------|
| `FIRECRAWL_API_KEY` | Firecrawl API key | [firecrawl.dev/app/api-keys](https://www.firecrawl.dev/app/api-keys) |
| `OPENAI_API_KEY` | OpenAI API key | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `FIRE_ENRICH_UNLIMITED` | `false` | Set to `true` to enable unlimited mode in production |
| `NODE_ENV` | `production` | Automatically set by Netlify (don't set manually) |

### Setting Variables

**Via Dashboard:**
1. Site settings → Environment variables
2. Add each variable with its value
3. Set scope (all deploys, production, branch, etc.)

**Via CLI:**
```bash
netlify env:set VARIABLE_NAME "value"
```

## Troubleshooting

### Build Fails

**Issue:** Build timeout or errors
**Solution:**
- Check build logs in Netlify dashboard
- Verify Node version is 18+
- Ensure all dependencies install correctly
- Check for TypeScript/ESLint errors

### API Routes Not Working

**Issue:** 404 errors on `/api/*` routes
**Solution:**
- Verify `@netlify/plugin-nextjs` is installed (auto-installed)
- Check that API routes are in `app/api/` directory
- Verify function timeouts are sufficient (26s for Pro)

### Environment Variables Not Working

**Issue:** Variables undefined in runtime
**Solution:**
- Verify variables are set in Netlify dashboard
- Check variable names match exactly (case-sensitive)
- Redeploy after adding variables
- Check scope settings (production vs preview)

### SSE Streaming Not Working

**Issue:** Real-time updates don't work
**Solution:**
- Netlify supports SSE, but test thoroughly
- Check function timeouts (long enrichments may timeout)
- Consider implementing chunked responses

### Function Timeout

**Issue:** Enrichment process times out
**Solution:**
- Free tier: 10-second timeout
- Pro tier: 26-second timeout (configured in netlify.toml)
- Consider:
  - Processing in smaller batches
  - Using background jobs with webhooks
  - Upgrading to Pro tier

### Build Time Too Long

**Issue:** Build exceeds timeout limit
**Solution:**
- Free tier: 15-minute build timeout
- Pro tier: 20-minute build timeout
- Optimize:
  - Use build cache
  - Exclude dev dependencies
  - Consider splitting into smaller builds

## Performance Optimization

### Build Optimization

- Netlify automatically caches `node_modules` between builds
- Use `.netlify` directory for custom cache configuration
- Monitor build logs to identify slow steps

### Runtime Optimization

- API routes run as serverless functions
- Static pages are pre-rendered and cached
- Dynamic routes are rendered on-demand

### Cost Considerations

- Free tier includes:
  - 100 GB bandwidth/month
  - 300 build minutes/month
  - 125,000 serverless function invocations/month
- Monitor usage in Netlify dashboard

## Custom Domain

1. Go to **Domain settings** → **Add custom domain**
2. Enter your domain name
3. Follow DNS configuration instructions
4. Netlify provides free SSL certificates automatically

## Continuous Deployment

With Git integration enabled:
- **Main branch:** Deploys to production
- **Pull requests:** Create preview deployments
- **Other branches:** Can be configured for branch-specific deployments

## Monitoring & Analytics

- **Netlify Analytics:** Built-in analytics (Pro tier)
- **Function logs:** View in Site settings → Functions
- **Build logs:** Available for each deployment

## Support

- [Netlify Documentation](https://docs.netlify.com)
- [Next.js on Netlify](https://docs.netlify.com/integrations/frameworks/next-js/)
- [Netlify Support](https://www.netlify.com/support)

## Next Steps

After successful deployment:
1. Set up a custom domain (optional)
2. Configure branch-specific environment variables if needed
3. Set up deployment notifications
4. Monitor function usage and optimize if needed
5. Consider setting up analytics


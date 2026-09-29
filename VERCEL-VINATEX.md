# LobeHub Vinatex Da Nang - Vercel Deployment

## Target architecture

Employee -> ai.vinatexdn.com.vn -> Vercel/LobeHub -> Neon PostgreSQL + managed Redis + S3-compatible storage -> AI providers.

This deployment is separate from the Docker Compose deployment. Do not use Docker hostnames such as `postgresql`, `redis`, `rustfs`, or `gateway` in Vercel environment variables.

## 1. Vercel project

Import `vinatexdng-cmd/lobehub` and deploy branch `vinatex-production`.

Keep the repository `vercel.json`. It currently defines:

- Build command: `bun run build:vercel`
- Install command: `npx pnpm@12.4.1 install`

Do not replace the build command with a plain `next build`, because the repository's Vercel build workflow is responsible for the application's expected build/database flow.

## 2. PostgreSQL

Create a Neon PostgreSQL production database and copy its connection string into the Vercel variable `DATABASE_URL`.

Required variables:

```env
DATABASE_DRIVER=node
DATABASE_URL=postgresql://...
KEY_VAULTS_SECRET=...
AUTH_SECRET=...
```

Never commit the real values to GitHub.

## 3. Redis

For multi-user production use, configure a managed Redis service and set:

```env
REDIS_URL=rediss://...
REDIS_PREFIX=lobehub
```

## 4. Knowledge Base / file storage

Use an S3-compatible object store such as Cloudflare R2 and configure:

```env
S3_ENDPOINT=...
S3_BUCKET=vinatex-lobehub
S3_ACCESS_KEY_ID=...
S3_SECRET_ACCESS_KEY=...
S3_ENABLE_PATH_STYLE=1
S3_SET_ACL=0
```

Verify CORS for browser uploads before enabling Knowledge Base for employees.

## 5. AI providers

Store provider keys only in Vercel Environment Variables:

```env
OPENAI_API_KEY=...
GOOGLE_API_KEY=...
DEEPSEEK_API_KEY=...
```

Do not expose provider API keys to employee browsers and do not commit them to this repository.

## 6. Authentication rollout

Pilot with IT accounts first. After authentication is verified, restrict employee access with the supported authentication configuration. The upstream environment template supports an allowed-email list and SSO-only mode.

Recommended rollout:

1. IT pilot.
2. VPCTY pilot users.
3. Phu My factory.
4. Nghia Hanh factory.
5. An Don factory.
6. Dung Quat factory.

## 7. Custom domain

After the Vercel deployment is healthy, add `ai.vinatexdn.com.vn` in Vercel Project > Settings > Domains and create the DNS record requested by Vercel.

Then set:

```env
APP_URL=https://ai.vinatexdn.com.vn
```

Redeploy after changing production environment variables.

## 8. First production checks

Confirm in this order:

- Vercel build succeeds.
- Database migrations complete.
- Home/login page loads.
- A pilot user can authenticate.
- A basic AI conversation succeeds.
- Redis connection is healthy.
- File upload works.
- Knowledge Base ingestion and retrieval work.
- Custom domain and HTTPS work.

## Security

- Never commit `.env` with real values.
- Rotate a key immediately if it is exposed in Git history, logs or screenshots.
- Use separate production and development credentials.
- Restrict database/storage credentials to the minimum permissions needed.
- Begin with a small employee pilot before company-wide rollout.

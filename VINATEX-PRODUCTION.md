# AI Vinatex Da Nang - LobeHub Production

This branch keeps upstream LobeHub deployment files intact and adds Vinatex-specific production guidance.

## Target architecture

Employee -> Cloudflare -> LobeHub -> PostgreSQL / Redis / RustFS -> AI providers

Public endpoints planned:

- `https://ai.vinatexdn.com.vn` -> LobeHub `localhost:3210`
- `https://files-ai.vinatexdn.com.vn` -> RustFS S3 `localhost:9000`
- `https://gateway-ai.vinatexdn.com.vn` -> Agent Gateway `localhost:8787` (WebSocket required)

Do not expose PostgreSQL 5432, Redis 6379, or RustFS Admin 9001 to the Internet.

## 1. Clone this branch

```bash
git clone -b vinatex-production https://github.com/vinatexdng-cmd/lobehub.git
cd lobehub/docker-compose/deploy
```

## 2. Create production environment

```bash
cp .env.vinatex.example .env
```

Replace every `CHANGE_ME` value locally. Never commit `.env`.

The upstream `setup.sh` can be used as a reference for generating LobeHub gateway/JWKS secrets. Keep the generated private JWKS key only on the server.

## 3. Configure AI providers

Set only server-side provider keys required by the company. Start with a small pilot model set. Do not distribute provider API keys to employees.

Recommended logical employee choices:

- AI Vinatex - Cong viec
- AI Vinatex - Nhanh
- AI Vinatex - Phan tich
- AI Vinatex - Tai lieu

Confirm the exact provider/model IDs supported by the deployed LobeHub version before enabling model aliases.

## 4. Start stack

```bash
docker compose pull
docker compose up -d
docker compose ps
```

Check logs:

```bash
docker compose logs -f lobe
docker compose logs -f postgresql
docker compose logs -f redis
docker compose logs -f rustfs
docker compose logs -f gateway
```

## 5. Cloudflare Tunnel

Create three public hostnames on the existing `vinatexdn` tunnel:

| Public hostname | Local service | Notes |
|---|---|---|
| `ai.vinatexdn.com.vn` | `http://localhost:3210` | Main employee portal |
| `files-ai.vinatexdn.com.vn` | `http://localhost:9000` | Browser file upload / knowledge files |
| `gateway-ai.vinatexdn.com.vn` | `http://localhost:8787` | Agent Gateway; WebSocket support required |

Do not publish database/cache/admin ports through Cloudflare.

## 6. Authentication rollout

Pilot phase:

1. Set `AUTH_ALLOWED_EMAILS` to IT/test users only.
2. Test chat, file upload, knowledge base and agent gateway.
3. Configure company SSO (prefer Microsoft Entra ID when available).
4. After SSO is verified, consider `AUTH_DISABLE_EMAIL_PASSWORD=1`.
5. Expand access by department/factory in controlled stages.

## 7. Security checklist

- [ ] `.env` exists only on production server.
- [ ] No API keys or real passwords are committed to GitHub.
- [ ] PostgreSQL 5432 is not Internet-accessible.
- [ ] Redis 6379 is not Internet-accessible.
- [ ] RustFS Admin 9001 is not Internet-accessible.
- [ ] Cloudflare HTTPS is active for all browser-facing endpoints.
- [ ] Agent Gateway WebSocket works through Cloudflare.
- [ ] Pilot login restrictions are enabled before employee rollout.
- [ ] Backup policy covers PostgreSQL and RustFS persistent data.
- [ ] Restore procedure is tested.

## 8. Rollout order

1. IT pilot
2. Office departments
3. Phu My factory
4. Nghia Hanh factory
5. An Don factory
6. Dung Quat factory

Do not connect sensitive HR/ERP data to a general employee knowledge base until document-level access controls and data classification are verified.

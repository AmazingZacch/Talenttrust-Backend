# TalentTrust Backend

## Live

- Website: https://talenttrust-frontend.vercel.app
- Docs: https://talenttrust-frontend.vercel.app/docs
- API: https://talenttrust-backend.vercel.app
- API docs (Swagger UI): https://talenttrust-backend.vercel.app/docs
- Contract (Stellar testnet): [`CCZMIVUBMG5SBC47SDTC32RWLBJ2HW2DCFWWJEOKCH2IFJBZGBKTXK5WH](https://stellar.expert/explorer/testnet/contract/CCZMIVUBMG5SBC47SDTC32RWLBJ2HW2DCFWWJEOKCH2IFJBZGBKTXK5WHY
 
Express + TypeScript API for TalentTrust, a milestone-based freelancer escrow protocol on Stellar.
The API mirrors the state machine of the Soroban escrow contract
([Talenttrust-Contracts](https://github.com/Talenttrust/Talenttrust-Contracts)) so clients and
freelancers can create engagements, fund escrow, release milestones, raise disputes and build reputation.

Storage is in-memory (seeded with demo data), so no database is required. On serverless hosting the
data is per-instance and resets on redeploy.

## Endpoints

| Method | Path | Description |
| --- | --- | --- |
| GET | `/` | Service info and endpoint list (JSON) |
| GET | `/health` | Health check |
| GET | `/docs` | Swagger UI |
| GET | `/openapi.json` | OpenAPI 3 spec |
| GET | `/api/v1/chain` | On-chain contract deployment info |
| GET | `/api/v1/stats` | Aggregate statistics |
| GET | `/api/v1/accounts/:address/readiness` | Check whether an account can fund an escrow (`token`, `amount` in base units) |
| GET/POST | `/api/v1/contracts` | List (`?status=`, `?party=`) / create escrow contracts |
| GET | `/api/v1/contracts/:id` | Get a contract |
| POST | `/api/v1/contracts/:id/deposit` | { "amount": 100 } |
| POST | `/api/v1/contracts/:id/milestones/:index/release` | Release a milestone |
| POST | `/api/v1/contracts/:id/dispute` | { "raisedBy": "G...", "reason": "..." } |
| POST | `/api/v1/contracts/:id/resolve` | { "freelancerBps": 5000 } |
| POST | `/api/v1/contracts/:id/cancel` | Cancel and refund |
| POST | `/api/v1/contracts/:id/reputation` | { "rating": 5 } |
| GET | `/api/v1/reputation/:address` | Freelancer reputation |

## Development

```bash
npm ci
npm run dev      # http://localhost:3001
npm run lint     # type check
npm test
npm run build && npm start
```

Configuration (all optional, see `.env.example`): `PORT`, `ESCROW_CONTRACT_ID`, `STELLAR_NETWORK`,
`STELLAR_RPC_URL`, `HORIZON_URL`, `FRONTEND_URL`, `ACCOUNT_READINESS_FEE_HEADROOM_XLM`,
`ACCOUNT_READINESS_CACHE_TTL_MS`.

## Account readiness

GET /api/v1/accounts/:address/readiness?token=<native | C...>&amount=<base units> inspects a

Stellar account via Horizon (and, for non-classic tokens, the token contract via Soroban RPC)
and reports whether it can fund a given amount before the user signs a deposit transaction.

Example:

```bash
curl -s 'localhost:3001/api/v1/accounts/GB.../readiness?token=native&amount=1000000000' | jq '{canFund, problems}'
```

Response fields: `address`, `exists`, `xlmBalance`, `minimumBalance`, `availableXlm`, `token`
`{contractId, kind, trustline, balance, authorized}`, `canFund`, `problems`. Problem codes are
`account_not_found`, `missing_trustline`, `not_authorized`, `insufficient_balance`, `insufficient_xlm_for_fees`.

## Deployment

Deployed on Vercel as a serverless function: `api/index.ts` exports the Express app and
vercel.json
rewrites every path to it. `src/index.ts` only calls `listen()` when run directly.

```bash
vercel deploy --prod
```

## License

MIT

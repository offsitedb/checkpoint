# OffsiteDB Checkpoint

**Never run a migration without a checkpoint again.**

This GitHub Action calls [OffsiteDB](https://offsitedb.com) to seal a **tagged, restore-drilled
snapshot** of your Postgres database — and blocks until it exists. If the snapshot (or its
restore drill) fails, the step fails, and your migration never runs against a database you
can't roll back.

## Usage

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Checkpoint database before migration
        uses: offsitedb/checkpoint@v1
        with:
          api-key: ${{ secrets.OFFSITEDB_API_KEY }}
          database: prod-api

      - name: Run migrations
        run: npm run migrate
```

That's the whole integration. The checkpoint step succeeds only after OffsiteDB has dumped,
encrypted, shipped, **and restore-drilled** the snapshot on a real Postgres cluster.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | yes | — | OffsiteDB API key (dashboard → Settings → API access). Store it as a repo secret. |
| `database` | yes | — | The database name or id as it appears in your OffsiteDB dashboard. |
| `url` | no | `https://offsitedb.com` | Base URL — override for self-hosted instances. |
| `tag` | no | `pre-deploy-<sha>` | Tag written into the snapshot ledger, so you can find "the one right before deploy 4f3a9c1". |
| `timeout-seconds` | no | `900` | How long to wait for the dump + restore drill. Raise for very large databases. |

## What a failure looks like

If the backup or its restore drill fails, the step exits non-zero with:

```
Error: Checkpoint failed — migration should not proceed
```

…and the response body, which includes OffsiteDB's plain-English failure diagnosis when
available. Your migration step never runs.

## Restoring a checkpoint

Every checkpoint is a standard `pg_dump` custom-format archive, restore-verified before it's
marked sealed. Find it in your OffsiteDB ledger by tag, download it decrypted (or pull it from
your own bucket), then:

```bash
pg_restore -d "$DATABASE_URL" --clean --if-exists pre-deploy-4f3a9c1.dump
```

---

Built by [OffsiteDB](https://offsitedb.com) — automated, encrypted, off-site Postgres backups
that prove they restore. · [Security](https://offsitedb.com/security) ·
[support@offsitedb.com](mailto:support@offsitedb.com)

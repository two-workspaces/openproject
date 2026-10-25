# Deploy and Host OpenProject on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/openproject-production?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=openproject)

[OpenProject](https://www.openproject.org) is open-source project management software, often used instead of Jira, MS Project or Asana. It covers work packages (tasks, bugs, features), Gantt charts, agile boards, backlogs, time and cost tracking, meetings, wikis and roadmaps. It is GPLv3 and has been developed since 2012.

## About Hosting OpenProject

This template runs OpenProject 17.9.3 from the official slim image, split the way upstream's production setup splits it:

| Service | What it does |
|---|---|
| **OpenProject** | The web app and API. Before each deploy it runs OpenProject's seeder, which migrates the database and creates the admin account and demo projects on first boot. |
| **Worker** | Background jobs: notifications, emails, attachment processing, recurring tasks. |
| **Postgres** | PostgreSQL 17 on a volume. |
| **Files** | A Railway bucket for attachments. Web and worker both read it, which a single-service volume can't do. |

What's set up for you:

- **A real admin password.** Other OpenProject templates leave the default `admin` / `admin` login, so anyone who finds the URL first can take over. Here the `admin` account gets a generated password, and you aren't forced to change it on first sign-in.
- **A real secret key.** `SECRET_KEY_BASE` is generated. A common mistake is setting `OPENPROJECT_SECRET_KEY_BASE`, which OpenProject ignores, so it falls back to a placeholder.
- **Pinned version.** Both services run `openproject/openproject:17.9.3-slim`, so a redeploy never upgrades you by surprise.
- **No SMTP needed to start.** You can sign in, create projects and add users without email set up.

## Common Use Cases

- Classic and agile project management for teams: Gantt timelines, Scrum backlogs, Kanban boards
- Replacing Jira or MS Project with a self-hosted tool your data stays on
- Time and cost tracking, meeting agendas and minutes, and project wikis in one place

## Dependencies for OpenProject Hosting

- PostgreSQL 17 and a Railway bucket (both included)
- SMTP credentials (optional) for email notifications and password resets

### Deployment Dependencies

- [OpenProject documentation](https://www.openproject.org/docs/)
- [OpenProject on GitHub](https://github.com/opf/openproject)
- [Template source on GitHub](https://github.com/two-workspaces/openproject)

### Implementation Details

**Sign in.** The first deploy takes a few minutes while the seeder sets up the database. Then open the OpenProject service's Railway domain and sign in as `admin`, with the `OPENPROJECT_SEED__ADMIN__USER__PASSWORD` value from the service's Variables tab. The variable is only used to create the account, so change the password afterwards under My account. Two sample projects ("Demo project" and "Scrum project") are included to explore. Delete them whenever you like.

**Add users.** Go to Administration → Users and permissions → Users → **+ User**. Without email set up, open the new user and set a password in the **Authentication** section of their General tab. The **Create a new account** link on the sign-in page is also on, but new registrations stay locked until an admin activates them. You can turn self-registration off under Administration → Authentication → Login and registration.

**Use the API.** Create a token under My account → Access tokens → API, then:

```bash
curl -u "apikey:<your-token>" "https://<your-openproject>.up.railway.app/api/v3/projects"
```

**Email (optional).** Add these to both services, then deploy:

- `OPENPROJECT_EMAIL__DELIVERY__METHOD=smtp`
- `OPENPROJECT_SMTP__ADDRESS`, `OPENPROJECT_SMTP__PORT`
- `OPENPROJECT_SMTP__USER__NAME`, `OPENPROJECT_SMTP__PASSWORD`
- `OPENPROJECT_SMTP__AUTHENTICATION=login`, `OPENPROJECT_SMTP__ENABLE__STARTTLS__AUTO=true`
- `OPENPROJECT_MAIL__FROM`

**Custom domain.** Add it to the OpenProject service, then set `OPENPROJECT_HOST__NAME` to the new host name (without `https://`) on both services and deploy.

**Upgrade.** Change the image tag on both OpenProject and Worker to the same new version, for example `openproject/openproject:17.10.0-slim`. The seeder runs the migrations before the new web version starts. Read the [release notes](https://www.openproject.org/docs/release-notes/) first.

**Not included.** Real-time collaborative editing of documents (it needs OpenProject's separate hocuspocus server), creating work packages from incoming email, and the BIM edition. Enterprise features show an "Enterprise edition" badge and need a license.

**Variables you might change.**

- `SECRET_KEY_BASE`: never change it after the first deploy.
- `OPENPROJECT_WEB_WORKERS` (default `0`, a single process with 8 threads): raise it to `2` or more on bigger plans for more throughput.
- `OPENPROJECT_DEFAULT__LANGUAGE`: the default language, for example `de` or `fr`.

**Memory.** Measured on Railway: about 1.65 GB idle in total (web about 800 MB, peaking near 1.4 GB right after a deploy; worker about 750 MB; Postgres about 80 MB). Use the Hobby plan or above.

## Why Deploy OpenProject on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying OpenProject on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.

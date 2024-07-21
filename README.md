# Healthchecks

Healthchecks is a self-hosted monitoring service for cron jobs, scheduled tasks, and background processes. Jobs report successful execution by sending HTTP requests or email pings to unique check endpoints. If an expected ping does not arrive on time, Healthchecks marks the check as late or down and sends notifications through configured integrations.

## Overview

Healthchecks provides a lightweight dead-man-switch model for monitoring tasks that do not expose their own metrics or health endpoints.

Each check can use either a simple period and grace time or a cron expression. Healthchecks records received pings, tracks current status, maintains an event log, and alerts team members when scheduled work stops running as expected.

The platform is suitable for:

- Cron job monitoring
- Database backup verification
- Scheduled report monitoring
- ETL and synchronization jobs
- Queue and batch worker checks
- Certificate renewal tasks
- Maintenance scripts
- Periodic infrastructure automation

## Features

- HTTP-based ping monitoring
- Email-based ping ingestion
- Period and grace-time schedules
- Cron-expression schedules
- Start, success, failure, and log signals
- Live check status dashboard
- Per-check event history
- Public check-status indicators with hard-to-guess identifiers
- Projects and team collaboration
- Read-only team access
- API access for checks and integrations
- Monthly and periodic email reports
- WebAuthn two-factor authentication
- TOTP-based authentication support
- Registration controls
- Remote-user authentication support
- Prometheus integration
- StatsD metrics support
- Optional S3-compatible storage for large ping bodies
- Outbound webhook notifications
- Email notifications
- Slack, Discord, Microsoft Teams, Telegram, Signal, Matrix, Mattermost, ntfy, Opsgenie, PagerDuty, Pushover, Rocket.Chat, Zulip, and other notification integrations
- Apprise integration for additional notification providers
- Docker-based deployment
- PostgreSQL, MySQL, MariaDB, and SQLite support

## Tech Stack

| Area | Technologies |
| --- | --- |
| Language | Python 3.12 or later |
| Web framework | Django 6.1 |
| Development database | SQLite |
| Production databases | PostgreSQL, MySQL, MariaDB |
| Application server | uWSGI |
| Static files | WhiteNoise |
| Asset compression | django-compressor |
| Authentication | Django authentication, WebAuthn, TOTP, JWT |
| Schedule evaluation | cronsim, oncalendar |
| HTTP integrations | PycURL |
| Email ingestion | aiosmtpd |
| Validation | Pydantic |
| Object storage | S3-compatible storage |
| Metrics | StatsD, Prometheus integration |
| Containerization | Docker, Docker Compose |
| Testing | Django test framework |
| Formatting | Ruff |

## Installation

### Development Setup

#### Requirements

- Python 3.12 or later
- Python virtual-environment support
- Git
- C compiler and Python development headers
- PostgreSQL development headers
- OpenSSL and libcurl development libraries

On Debian or Ubuntu:

```bash
sudo apt update
sudo apt install -y gcc python3-dev python3-venv libpq-dev libcurl4-openssl-dev libssl-dev
```

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install wheel
```

Install dependencies:

```bash
pip install -r requirements.txt -r requirements-dev.txt
```

Apply database migrations:

```bash
./manage.py migrate
```

Create an administrator:

```bash
./manage.py createsuperuser
```

Start the development server:

```bash
./manage.py runserver
```

The default development configuration uses SQLite and listens on port `8000`.

### Docker Compose

Create the Docker environment file:

```bash
cd docker
cp .env.example .env
```

At minimum, configure:

```env
ALLOWED_HOSTS=your-healthchecks-host
DEFAULT_FROM_EMAIL=healthchecks@example.org

EMAIL_HOST=your-smtp-host
EMAIL_HOST_USER=your-smtp-user
EMAIL_HOST_PASSWORD=your-smtp-password

SECRET_KEY=replace-with-a-long-random-value
SITE_ROOT=your-public-instance-origin
```

Replace the example PostgreSQL password before starting the stack.

Start the services:

```bash
docker compose up -d
```

The reference stack starts PostgreSQL and the Healthchecks web application.

The application is exposed on port `8000`.

Create an administrator:

```bash
docker compose run web /opt/healthchecks/manage.py createsuperuser
```

View logs:

```bash
docker compose logs -f
```

Stop the services:

```bash
docker compose down
```

The reference Docker deployment does not terminate TLS. Production installations should use a trusted reverse proxy or load balancer.

## Usage

### Create a Check

1. Sign in to the dashboard.
2. Create a check.
3. Choose a simple period and grace time or define a cron schedule.
4. Copy the generated ping identifier.
5. Add a ping command to the monitored job.
6. Configure one or more notification channels.
7. Test both successful and failed execution paths.

### Basic Ping

A monitored process can report success with an HTTP request to its unique ping endpoint:

```bash
curl -fsS --retry 3 "$PING_ENDPOINT/<check-uuid>"
```

### Signal Job Start

```bash
curl -fsS "$PING_ENDPOINT/<check-uuid>/start"
```

### Signal Failure

```bash
curl -fsS "$PING_ENDPOINT/<check-uuid>/fail"
```

### Shell Example

```bash
backup-command && curl -fsS "$PING_ENDPOINT/<check-uuid>"
```

For production scripts, preserve the original command exit status and handle network failures according to the job requirements.

### Cron Scheduling

Checks can use cron expressions instead of fixed periods.

Example:

```text
15 2 * * *
```

The scheduler evaluates the configured expression and grace period to determine when a missing ping should transition the check into a late or down state.

## Configuration

Healthchecks reads most runtime settings from environment variables. Settings can also be overridden through `hc/local_settings.py`.

### Core Settings

| Variable | Purpose |
| --- | --- |
| `SECRET_KEY` | Django signing and session secret |
| `DEBUG` | Enables development debugging |
| `SITE_ROOT` | Public application origin |
| `SITE_NAME` | Instance display name |
| `SITE_LOGO_URL` | Optional custom logo location |
| `ALLOWED_HOSTS` | Accepted hostnames |
| `REGISTRATION_OPEN` | Enables public user registration |
| `REMOTE_USER_HEADER` | Enables header-based authentication |
| `RP_ID` | WebAuthn relying-party identifier |
| `PING_ENDPOINT` | Base endpoint for incoming HTTP pings |
| `PING_EMAIL_DOMAIN` | Domain used for email-based pings |
| `PING_BODY_LIMIT` | Maximum stored ping-body size |
| `METRICS_KEY` | Access key for protected metrics |
| `STATSD_HOST` | StatsD server address |

### Database Settings

| Variable | Purpose |
| --- | --- |
| `DB` | Selects PostgreSQL, MySQL, or MariaDB |
| `DB_HOST` | Database host |
| `DB_PORT` | Database port |
| `DB_NAME` | Database name |
| `DB_USER` | Database user |
| `DB_PASSWORD` | Database password |
| `DB_CONN_MAX_AGE` | Persistent connection lifetime |
| `DB_SSLMODE` | PostgreSQL SSL mode |
| `DB_TARGET_SESSION_ATTRS` | PostgreSQL target-session policy |

When `DB` is not configured, Healthchecks uses SQLite.

### Email Settings

| Variable | Purpose |
| --- | --- |
| `DEFAULT_FROM_EMAIL` | Default sender address |
| `SUPPORT_EMAIL` | Support contact address |
| `EMAIL_HOST` | SMTP server |
| `EMAIL_PORT` | SMTP port |
| `EMAIL_HOST_USER` | SMTP username |
| `EMAIL_HOST_PASSWORD` | SMTP password |
| `EMAIL_USE_TLS` | Enables SMTP TLS |
| `EMAIL_USE_SSL` | Enables SMTP SSL |
| `EMAIL_USE_VERIFICATION` | Enables certificate verification |
| `EMAIL_MAIL_FROM_TMPL` | Template for outbound sender addresses |

### Object Storage

Large ping bodies can optionally use S3-compatible object storage.

| Variable | Purpose |
| --- | --- |
| `S3_ACCESS_KEY` | Object-storage access key |
| `S3_SECRET_KEY` | Object-storage secret |
| `S3_ENDPOINT` | Object-storage endpoint |
| `S3_REGION` | Storage region |
| `S3_BUCKET` | Bucket name |
| `S3_TIMEOUT` | Request timeout |
| `S3_SECURE` | Enables secure object-storage transport |

## Production Deployment

A production installation requires more than the Django web process.

### Runtime Processes

Run the application through a production WSGI server such as uWSGI.

The alert process must remain continuously available:

```bash
./manage.py sendalerts
```

For periodic reports and reminders:

```bash
./manage.py sendreports --loop
```

If email-based pings are enabled, run the SMTP receiver with the required port configuration.

The provided Docker image starts the supporting Healthchecks processes alongside uWSGI.

### Release Tasks

Run these during application upgrades:

```bash
./manage.py migrate
./manage.py collectstatic --noinput
./manage.py compress
```

Production deployments should:

- Set `DEBUG=False`
- Generate a unique `SECRET_KEY`
- Restrict `ALLOWED_HOSTS`
- Use PostgreSQL, MySQL, or MariaDB for larger deployments
- Secure and back up the database
- Configure outbound SMTP
- Run behind HTTPS
- Use a trusted reverse proxy
- Configure forwarded client addresses correctly
- Keep `sendalerts` continuously running
- Run `sendreports --loop` when reports are required
- Monitor the Healthchecks instance itself
- Protect integration credentials
- Persist database and object-storage data
- Test alerts after infrastructure changes

## API

Healthchecks exposes API operations for programmatically managing monitoring resources.

Typical automation workflows include:

- Creating checks
- Listing checks
- Updating schedules
- Pausing or deleting checks
- Managing notification channels
- Retrieving check state

Use API credentials with the minimum required access and avoid placing them in source-controlled scripts.

## Contributing

Install runtime and development dependencies:

```bash
pip install -r requirements.txt -r requirements-dev.txt
```

Run the test suite:

```bash
./manage.py test
```

Format and validate Python changes with Ruff.

Render documentation changes with:

```bash
./manage.py render_docs
```

Before making a substantial upstream change:

- Discuss larger features before implementation
- Add tests for bug fixes and new behavior
- Prefer simple, maintainable implementations
- Follow existing integration patterns
- Update documentation for user-facing changes
- Keep credentials and deployment secrets out of commits

The upstream repository currently does not accept content generated or edited with AI tools. Pull requests are also disabled because of AI-generated spam, so completed upstream contributions are coordinated through repository issues. Follow the current contribution policy before submitting work.

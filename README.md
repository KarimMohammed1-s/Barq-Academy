 BARQ Academy DevOps Assessment

This repository contains the implementation, investigation evidence, validation
scripts, failure testing, persistence testing, backup/restore procedures,
security review, engineering decisions, and CI configuration for the BARQ  DevOps assessment.

## Current verified environment

The currently verified environment contains:

- NGINX as the only public entry point
- Flask application instances `app-01` and `app-02`
- PostgreSQL 16
- Redis 7.4 with AOF persistence
- Separate frontend and backend Docker networks
- Internal backend network
- PostgreSQL named-volume persistence
- Redis named-volume persistence
- Application readiness checks
- Container health checks
- Restart policies
- Resource limits
- Non-root application container
- Pinned container image digests
- No host ports for applications, PostgreSQL, or Redis

Current verified development endpoint:

    http://127.0.0.1:8080

The final assessment video will demonstrate the required three-instance
configuration on port 8090.

## Architecture

Current request flow:

    Client
      |
      v
    NGINX :80
      |
      +---- app-01:8080
      |
      +---- app-02:8080
                  |
                  +---- PostgreSQL :5432
                  |
                  +---- Redis :6379

Networks:

- frontend: NGINX and application containers
- backend: application containers, PostgreSQL, and Redis
- backend is an internal Docker network

PostgreSQL and Redis do not expose host ports.

See `docs/ARCHITECTURE.md` and the final `architecture.png` or
`architecture.pdf`.

## Repository structure

    app/                    Flask application
    database/               PostgreSQL initialization
    nginx/                  NGINX configuration
    logs/                   Original assessment logs
    tests/                  Application tests
    docs/                   Documentation and evidence
    assessment/             Assessment material
    assets/                 Assessment assets

    docker-compose.yml      Docker Compose configuration
    Dockerfile              Application image
    requirements.txt        Python dependencies

    validate.py             Environment validation
    failure_test.py         Backend failure/recovery test
    backup.sh               PostgreSQL backup
    restore.sh              PostgreSQL restore
    video_challenge.sh      Final live troubleshooting challenge

    log_analysis.md         Correlated log analysis
    troubleshooting.md      Investigation journal
    decisions.md            Engineering decisions
    security_review.md      Security review
    AI_USAGE.md             AI/tool usage disclosure

## Prerequisites

Install or have available:

- Ubuntu/Linux
- Docker Engine
- Docker Compose v2
- Git
- Python 3

Verify:

    git --version
    docker version
    docker compose version
    python3 --version

Docker commands may require `sudo` depending on local Docker permissions.


## HTTP endpoints

Required endpoints:

    /
    /health
    /ready
    /instance
    /records
    /counter

Test them:

    curl -i http://127.0.0.1:8080/
    curl -i http://127.0.0.1:8080/health
    curl -i http://127.0.0.1:8080/ready
    curl -i http://127.0.0.1:8080/instance
    curl -i http://127.0.0.1:8080/records
    curl -i http://127.0.0.1:8080/counter

Repeated `/instance` requests demonstrate load balancing:

    for i in {1..10}; do
        curl -s http://127.0.0.1:8080/instance
        echo
    done

## Automated validation

Run:

    ./validate.py

The validator checks:

- Container health
- Public NGINX access
- Required HTTP endpoints
- Application readiness
- PostgreSQL readiness
- Redis readiness
- Load balancing
- Host port exposure
- Network isolation
- Internal backend network
- PostgreSQL records
- Redis counter

The verified environment currently ends with:

    === VALIDATION PASSED ===
    All required checks passed.

## Failure and recovery test

Run:

    ./failure_test.py

The test:

1. Measures baseline traffic.
2. Stops `app-01`.
3. Sends traffic through NGINX.
4. Confirms `app-02` continues serving requests.
5. Restores `app-01`.
6. Waits for it to become healthy.
7. Confirms traffic returns to both instances.

The verified run achieved:

    Baseline:       20/20 successful
    During failure: 20/20 successful
    Errors:          0/20
    After recovery: 20/20 successful

The analysis covers:

- Log validity
- Status-code counts
- Duplicate request IDs
- NGINX retry behavior
- Dependency failures
- Upstream failures
- Timeout incidents
- Latency
- Cross-log correlation
- Timeline
- Root-cause conclusions

## Security review

The security review covers:

- Secrets
- Host ports
- Container users
- Image pinning
- Docker networks
- Persistence and backups
- Logging
- Availability
- Resource limits

See:

    security_review.md

Implemented fixes are separated from future recommendations.

## CI

The GitHub Actions workflow is:

    .github/workflows/ci.yml

The pipeline is intended to run on pushes and pull requests and performs:

1. Checkout
2. Python syntax checks
3. Docker Compose configuration validation
4. Image build
5. Environment startup
6. Readiness wait
7. Full validation

The final successful CI run will be recorded in the evidence index.

## Final video challenge

The assessment requires a live troubleshooting demonstration using:

    ./video_challenge.sh

The challenge must be run for the first time in the video working copy.


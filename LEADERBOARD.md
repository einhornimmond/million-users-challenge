# Leaderboard

Every submission, scored on the same boxes with the same load test. New PRs are benchmarked as they come in and
added here, so this is the only leaderboard.

**The machines.** The app box is a Synteq VPS with 16 dedicated vCPUs (Xeon Platinum 8260) and 32 GB RAM on
Ubuntu 24.04, set up with [`box/setup-app-box.sh`](box/setup-app-box.sh) and nothing else. Postgres 18 runs on its own
16-core box with [`box/postgresql.conf`](box/postgresql.conf). Load comes from separate 16-core load generators in
the same datacenter, running [`bench/mload`](bench) with `-gzip -think-scale 2.5`.

**How each entry is scored.** Built from source with its own `install.sh` (root) and `build.sh` (user `challenge`),
checked with the 42-check test suite, then started by systemd as `challenge` on a fresh copy of the seed for every
run. Two-minute runs climb (each passing run jumps to where its CPU says ~85% would be), then narrow down to within 5%; a 5-minute hold confirms the
number, stepping down 5% until one passes (p95 under 500 ms, p99 under 1 s, under 1% errors). **Each entry gets the
most users held for 5 minutes on 16 cores, on 8 and on 4**. 16 cores is capped at 1,000,000 (past that the one
Postgres box is the limit); 8 and 4 cores are searched up to 3,000,000. `(max)` = held the search's ceiling. Ranked
by 16 cores.

| # | Submission | PR | Language | Stack | 16 cores | 8 cores | 4 cores | Scored |
|---:|---|---|---|---|---:|---:|---:|---|
| 1 | [`c-epoll-unorig-username`](submissions/c-epoll-unorig-username) | [#10](https://github.com/arjaythedev/million-users-challenge/pull/10) | C | own epoll event loop, hand-written HTTP/1.1, static libpq in pipeline mode, prepared statements, ISA-L gzip | 1,000,000 (max) | 1,420,000 | 1,370,000 | 2026-10-29 |
| 2 | [`java-epoll-stohirov`](submissions/java-epoll-stohirov) | [#2](https://github.com/arjaythedev/million-users-challenge/pull/2) | Java | own epoll event loop per core (FFM), hand-written HTTP/1.1 and Postgres wire client, pipelined, group commit | 1,000,000 (max) | 1,310,000 | 1,260,000 | 2026-10-28 |
| 3 | [`zig-iouring-harshitkhandelwal208`](https://github.com/arjaythedev/million-users-challenge/pull/4)\* | [#4](https://github.com/arjaythedev/million-users-challenge/pull/4) | Zig | own io_uring HTTP/1.1 and Postgres wire client, own gzip, read packs + group-commit writes | 1,000,000 (max) | 1,270,000 | 1,320,000 | 2026-10-29 |
| 4 | [`rust-tokio-unorig-username`](submissions/rust-tokio-unorig-username) | [#11](https://github.com/arjaythedev/million-users-challenge/pull/11) | Rust | current-thread tokio per core with `SO_REUSEPORT`, prepared statements, pipelined tokio-postgres | 1,000,000 (max) | 1,260,000 | 960,000 | 2026-10-29 |
| 5 | [`c-iouring-einhornimmond`](submissions/c-iouring-einhornimmond) | [#12](https://github.com/arjaythedev/million-users-challenge/pull/12) | C | own io_uring event loop (provided buffers, direct descriptors), libpq pipeline mode, ISA-L gzip | 1,000,000 (max) | 1,210,000 | 1,300,000 | 2026-10-29 |
| 6 | [`rust-tokio-cknutson12`](submissions/rust-tokio-cknutson12) | [#1](https://github.com/arjaythedev/million-users-challenge/pull/1) | Rust | thread-per-core tokio, hand-written HTTP/1.1, pipelined tokio-postgres | 1,000,000 (max) | 1,210,000 | 1,130,000 | 2026-10-28 |
| 7 | [`c-h2o-einhornimmond`](submissions/c-h2o-einhornimmond) | [#9](https://github.com/arjaythedev/million-users-challenge/pull/9) | C | h2o (libh2o) built with Zig, libpq pipeline mode, prepared statements | 1,000,000 (max) | 1,180,000 | 1,160,000 | 2026-10-29 |
| 8 | [`typescript-bun-tbsvttr`](submissions/typescript-bun-tbsvttr) | [#13](https://github.com/arjaythedev/million-users-challenge/pull/13) | TypeScript | Bun.serve with `reusePort`, a worker per core, Bun.SQL prepared statements, libdeflate gzip | 1,000,000 (max) | 1,080,000 | 395,000 | 2026-10-29 |
| 9 | [`c-iouring-TheFirstIstari`](https://github.com/arjaythedev/million-users-challenge/pull/16)† | [#16](https://github.com/arjaythedev/million-users-challenge/pull/16) | C | own io_uring/epoll event loop with `SO_REUSEPORT`, picohttpparser, libpq prepared statements, libdeflate gzip | 910,000 | 840,000 | 705,000 | 2026-10-30 |
| 10 | [`bun-elysia-youssefsz`](submissions/bun-elysia-youssefsz) | [#14](https://github.com/arjaythedev/million-users-challenge/pull/14) | TypeScript | Elysia 1.4 + Drizzle ORM over Bun SQL, prepared statements, admission queue | 840,000 | 500,000 | 225,000 | 2026-10-29 |
| 11 | [`java-quarkus-stohirov`](submissions/java-quarkus-stohirov) | [#3](https://github.com/arjaythedev/million-users-challenge/pull/3) | Java | Quarkus 3.40 (Vert.x Web router) + Vert.x reactive Postgres client | 780,000 | 605,000 | 440,000 | 2026-10-28 |
| 12 | [`csharp`](examples/csharp) (the video's server) | — | C# | ASP.NET Core + Npgsql | 665,000 | 540,000 | 300,000 | 2026-10-19 |
| 13 | [`elixir-bandit-tbsvttr`](submissions/elixir-bandit-tbsvttr) | [#15](https://github.com/arjaythedev/million-users-challenge/pull/15) | Elixir | Bandit 1.14 + bare Plug on OTP 29, Postgrex prepared statements | 650,000 | 315,000 | 129,000 | 2026-10-29 |
| 14 | [`node`](examples/node) (the video's server) | — | Node | Fastify 5 + node-postgres, `node:cluster` workers | 587,000 | 310,000 | 125,000 | 2026-10-19 |
| 15 | [`forth-gforth-FantasticForth`](submissions/forth-gforth-FantasticForth) | [#8](https://github.com/arjaythedev/million-users-challenge/pull/8) | Forth | Gforth, forked epoll workers with `SO_REUSEPORT`, libpq prepared statements via FFI | 420,000 | 290,000 | 158,000 | 2026-10-29 |
| 16 | [`rust`](examples/rust) (the video's server) | — | Rust | axum + SQLx | 413,000 | 227,000 | 114,000 | 2026-10-19 |
| 17 | [`java`](examples/java) (the video's server) | — | Java | Spring Boot 3.5 (Tomcat, virtual threads) + HikariCP + pgjdbc | 321,000 | 312,000 | 176,000 | 2026-10-19 |
| 18 | [`go`](examples/go) (the video's server) | — | Go | Gin + pgx/v5 `pgxpool` | 248,000 | 230,000 | 124,000 | 2026-10-19 |
| 19 | [`python`](examples/python) (the video's server) | — | Python | FastAPI on Gunicorn/Uvicorn workers + asyncpg | 213,000 | 104,000 | 51,000 | 2026-10-19 |
| 20 | [`cmake-lots_of_things-IdkWhatToWriteHereOhManItWasAlreadyUsed`](https://github.com/arjaythedev/million-users-challenge/pull/17)ᵖ | [#17](https://github.com/arjaythedev/million-users-challenge/pull/17) | CMake | libevent listener + NASM launcher running `cmake -P` per request (HTTP, JWT, JSON in CMake script), libpq pool daemon over a unix socket | 1,000 | 1,000 | 1,000 | 2026-10-30 |

The six example servers were scored for the video (2026-10-18/19) with borrowed load generators and the old doubling
search; the test, the boxes and the pass rules are otherwise the same.

\* Held, not merged yet: #4 accepts a JWT with no `exp`, which the SPEC requires. Scored at 1f3ac0c on kernel
6.8.0-117 (its earlier `failed` was our app box's 6.8.0-147 kernel, not the entry).

† Held, not merged yet: #16 ships its libdeflate and picohttpparser sources as tarballs in `vendor/`, still to be
checked against upstream. Scored at 7c3ad7d (the `404 post not found` bug of its first run is fixed).

ᵖ Held, not merged yet: #17's `install.sh` was patched for the run only (it chowns `/app` to root, so `build.sh` as
`challenge` couldn't write there; the run used `chown -R challenge:challenge /app`), and it installs into `/app` and
`/tmp` instead of its own folder. Scored at 79c8ed4.

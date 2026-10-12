FROM ubuntu:24.04

ARG RUNNER_VERSION=2.338.0
ARG RUNNER_SHA256=af4b794c1bc41d73d40535e3fe092a39f9679cd8d965954c2aca25a05ca41d32
# Keep in sync with playwright-core in bun.lock.
ARG PLAYWRIGHT_VERSION=1.63.0
ARG BUN_VERSION=1.4.2

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      ca-certificates curl git jq unzip xz-utils tar ffmpeg python3 tini \
 && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --uid 1001 --shell /bin/bash runner \
 && mkdir -p /home/runner/actions-runner \
 && curl -fsSL -o /tmp/runner.tar.gz \
      "https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz" \
 && echo "${RUNNER_SHA256}  /tmp/runner.tar.gz" | sha256sum -c - \
 && tar xzf /tmp/runner.tar.gz -C /home/runner/actions-runner \
 && rm /tmp/runner.tar.gz \
 && apt-get update \
 && /home/runner/actions-runner/bin/installdependencies.sh \
 && rm -rf /var/lib/apt/lists/* \
 && chown -R runner:runner /home/runner

# Chromium system libraries are installed as root at build time so jobs never need sudo.
# Bun is only a temporary tool to run playwright's install-deps and is removed afterwards.
RUN curl -fsSL https://bun.sh/install | BUN_INSTALL=/tmp/bun bash -s "bun-v${BUN_VERSION}" \
 && /tmp/bun/bin/bunx playwright-core@${PLAYWRIGHT_VERSION} install-deps chromium \
 && rm -rf /tmp/bun /root/.bun /root/.cache /var/lib/apt/lists/*

COPY entrypoint.sh /entrypoint.sh
RUN chmod 0755 /entrypoint.sh

USER runner
WORKDIR /home/runner/actions-runner
ENTRYPOINT ["tini", "--", "/entrypoint.sh"]

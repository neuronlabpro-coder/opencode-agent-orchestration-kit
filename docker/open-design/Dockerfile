FROM node:24-bookworm-slim

RUN apt-get update \
  && apt-get install -y --no-install-recommends git curl ca-certificates python3 make g++ \
  && rm -rf /var/lib/apt/lists/*

RUN useradd -m -s /bin/bash opencode
WORKDIR /app

ARG OPEN_DESIGN_REF=main
ARG OPENCODE_AI_VERSION=1.14.31

RUN git clone https://github.com/nexu-io/open-design.git /app/open-design \
  && cd /app/open-design \
  && git checkout "$OPEN_DESIGN_REF"
WORKDIR /app/open-design

RUN corepack enable \
  && corepack prepare pnpm@10.33.2 --activate \
  && pnpm install --frozen-lockfile=false

# 1. Compilar dependencias internas del monorepo
RUN pnpm rebuild || true

# 2. Parchear el host hardcodeado en el código fuente antes de compilar
RUN find apps/daemon/src -type f -name "*.ts" | xargs grep -l "127.0.0.1\|localhost" | while read f; do \
      sed -i "s/127\.0\.0\.1/process.env.OD_HOST || '0.0.0.0'/g" "$f"; \
      sed -i "s/'localhost'/process.env.OD_HOST || '0.0.0.0'/g" "$f"; \
    done || true

# 3. Compilar daemon
RUN cd apps/daemon && pnpm build

# 4. Compilar web
RUN cd apps/web && pnpm build

RUN npm install -g "opencode-ai@$OPENCODE_AI_VERSION"

ENV OD_HOST=0.0.0.0
ENV OD_PORT=7456
ENV OPENCODE_CONFIG_DIR=/home/opencode/.config/opencode

RUN mkdir -p /home/opencode/.config/opencode \
             /home/opencode/.local/share/opencode \
             /app/open-design/.od/projects \
  && chown -R opencode:opencode /home/opencode /app/open-design

USER opencode
EXPOSE 7456

CMD ["sh", "-lc", "node apps/daemon/dist/cli.js --port ${OD_PORT:-7456} --no-open"]

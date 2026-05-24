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
  && corepack prepare pnpm@10.33.2 --activate

# Instalar sin postinstall para evitar symlink roto del daemon
RUN pnpm install --frozen-lockfile=false --ignore-scripts

# 1. Compilar daemon primero
RUN cd apps/daemon && pnpm build

# 2. Compilar web después
RUN cd apps/web && pnpm build

# 3. Ejecutar postinstall ahora que daemon y web están compilados
RUN pnpm rebuild || true

RUN npm install -g "opencode-ai@$OPENCODE_AI_VERSION"

ENV OD_HOST=0.0.0.0
ENV OD_PORT=7456
ENV OPENCODE_CONFIG_DIR=/home/opencode/.config/opencode

RUN mkdir -p /home/opencode/.config/opencode /home/opencode/.local/share/opencode /app/open-design/.od \
  && chown -R opencode:opencode /home/opencode /app/open-design

USER opencode
EXPOSE 7456

# El daemon sirve el frontend compilado directamente
CMD ["sh", "-lc", "node apps/daemon/dist/cli.js --port ${OD_PORT:-7456} --no-open"]

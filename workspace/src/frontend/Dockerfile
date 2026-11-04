# syntax=docker/dockerfile:1.7

# -----------------------------------------------------------------------------
# Stage 1 - dependencies
# -----------------------------------------------------------------------------

FROM node:22-alpine AS deps

WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev --no-audit --no-fund
# --omit=dev: Bỏ thư viện devDependencies do này production.
# --no-audit: Tắt tính năng quyết lỗ hổng bảo mật khi cài.
# --no-fund Tắt thông báo kêu quyền góp tài trợ.

# -----------------------------------------------------------------------------
# Stage 2 - runtime
# -----------------------------------------------------------------------------

FROM node:22-alpine AS runtime

LABEL org.opencontainers.image.title="ivolve-frontend" \
      org.opencontainers.image.description="iVolve DevOps Roadmap - web frontend"
# Gắn metadata vào image theo chuẩn OCI - thêm thông tin.

RUN apk add --no-cache tini

ENV NODE_ENV=production \
    PORT=3000

WORKDIR /app

COPY --chown=node:node --from=deps /app/node_modules ./node_modules
COPY --chown=node:node package*.json ./
COPY --chown=node:node server.js ./
COPY --chown=node:node views ./views
COPY --chown=node:node public ./public

USER node
EXPOSE 3000

HEALTHCHECK --interval=15s --timeout=5s --start-period=20s --retries=3 \
    CMD node -e "require('http').get('http://127.0.0.1:3000/',r=>process.exit(r.statusCode<500?0:1)).on('error',()=>process.exit(1))"
    
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
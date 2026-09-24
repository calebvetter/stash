FROM node:24-alpine AS frontend

RUN apk add --no-cache make git

COPY ./ui/v2.5/package.json ./ui/v2.5/pnpm-lock.yaml /stash/ui/v2.5/

WORKDIR /stash

COPY Makefile /stash/
COPY ./graphql /stash/graphql/
COPY ./ui /stash/ui/

RUN npm install -g pnpm
RUN make pre-ui
RUN make generate-ui

ARG GITHASH
ARG STASH_VERSION

RUN BUILD_DATE=$(date +"%Y-%m-%d %H:%M:%S") make ui-only


FROM golang:1.24.3-alpine AS backend

RUN apk add --no-cache make alpine-sdk

WORKDIR /stash

COPY ./go* ./*.go Makefile gqlgen.yml .gqlgenc.yml /stash/
COPY ./graphql /stash/graphql/
COPY ./scripts /stash/scripts/
COPY ./pkg /stash/pkg/
COPY ./cmd /stash/cmd/
COPY ./internal /stash/internal/
COPY ./ui /stash/ui/

RUN make generate-backend generate-login-locale

COPY --from=frontend /stash /stash/

ARG GITHASH
ARG STASH_VERSION

RUN make flags-release flags-pie stash


# Preserve the exact official v0.31.1 runtime environment.
FROM stashapp/stash:v0.31.1

COPY --from=backend /stash/stash /usr/bin/stash

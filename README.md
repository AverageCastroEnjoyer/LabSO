#!/bin/sh
set -eu

APP_DIR="/opt/node-hello"
BRANCH="deploy"

as_oper() {
  runuser -u oper -- sh -c "cd '$APP_DIR' && $1"
}

as_oper "git fetch origin '$BRANCH'"

LOCAL_REV="$(as_oper "git rev-parse HEAD")"
REMOTE_REV="$(as_oper "git rev-parse 'origin/$BRANCH'")"

if [ "$LOCAL_REV" = "$REMOTE_REV" ]; then
  echo "Sin cambios nuevos en origin/$BRANCH."
  exit 0
fi

echo "Actualizando $APP_DIR desde origin/$BRANCH..."
as_oper "git checkout '$BRANCH'"
as_oper "git pull --ff-only origin '$BRANCH'"
as_oper "npm ci --omit=dev"
systemctl restart hello-node.service
echo "hello-node actualizado y reiniciado."

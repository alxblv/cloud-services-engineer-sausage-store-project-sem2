## Напишите multi-stage Dockerfile для фронтенда:
## на первом шаге будет собираться статика,
## а на втором выполните её копирование в образ nginx

FROM node:16-alpine3.18 AS building-frontend-app

WORKDIR /frontend

# copy both 'package.json' and 'package-lock.json' (if available)
COPY package*.json ./

# install project dependencies
RUN npm install

# copy project files and folders to the current working directory
COPY . .

# build the app
RUN npm run build

# deploy the application binary into a lean image
FROM nginx:1.31.1-alpine3.23-slim AS deploying-frontend-app

# curl will be used in HEALTHCHECK
RUN apk add --no-cache curl

# copy previously built app
COPY --from=building-frontend-app /frontend/dist/frontend  /usr/share/nginx/html/

# creating non-root user and changing owner (alpine)
RUN addgroup -S appuser && adduser -S appuser -G appuser
RUN chown -R appuser:appuser /usr/share/nginx/html /etc/nginx/conf.d /var/cache/nginx /run

# switching to a non-root user
USER appuser

EXPOSE 80

# without -f flag curl will return 0 even if HTTP status code is 4xx/5xx
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost/

# daemon off - so the main process is active and container does not exit immediately
CMD ["nginx", "-g", "daemon off;"]


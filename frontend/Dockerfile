FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN apk add --no-cache jq && jq 'del(.scripts.prepare)' package.json > package.json.tmp && mv package.json.tmp package.json && npm install

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

FROM node:20-alpine AS build
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:stable-alpine

COPY default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist/frontend/browser/ /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

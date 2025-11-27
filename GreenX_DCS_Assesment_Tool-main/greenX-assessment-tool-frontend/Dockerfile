# --------------------------
# 1) BUILD REACT APP
# --------------------------
FROM node:18 AS build
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install --legacy-peer-deps

COPY . .
RUN npm run build

# --------------------------
# 2) SERVE WITH NGINX
# --------------------------
FROM nginx:latest

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 8002

CMD ["nginx", "-g", "daemon off;"]

# ===============================
# 1) BUILD STAGE
# ===============================
FROM node:18 AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
RUN npm run build


# ===============================
# 2) DEPLOY STAGE WITH NGINX
# ===============================
FROM nginx:alpine

# Copy build output to nginx directory
COPY --from=builder /app/build /usr/share/nginx/html

# Expose frontend port
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

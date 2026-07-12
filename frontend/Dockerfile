# ---------- Stage 1: Build ----------
   FROM node:20-alpine AS builder

   WORKDIR /build

   COPY package.json npm-shrinkwrap.json ./
   RUN npm ci

   COPY . .
   RUN npm run build

   # ---------- Stage 2: Run ----------
   FROM nginx:alpine

   COPY nginx.conf /etc/nginx/conf.d/default.conf

   COPY --from=builder /build/dist/frontend/browser /usr/share/nginx/html

   EXPOSE 80
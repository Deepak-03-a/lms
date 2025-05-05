FROM node:16 AS lms-fe
WORKDIR /webapp
COPY ..
RUN npm install
RUN npm run build
FROM nginx
COPY --from=lms-fe /frontend/dist /usr/share/nginx/html
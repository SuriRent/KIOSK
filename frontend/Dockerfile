FROM node:22-alpine AS build

ARG REACT_APP_BACKEND_URL=
ARG REACT_APP_MARKETING_HOST=
ARG REACT_APP_APP_URL=
ENV REACT_APP_BACKEND_URL=$REACT_APP_BACKEND_URL \
    REACT_APP_MARKETING_HOST=$REACT_APP_MARKETING_HOST \
    REACT_APP_APP_URL=$REACT_APP_APP_URL

WORKDIR /app/frontend

COPY frontend/package.json ./
RUN npm install

COPY frontend/ ./
RUN npm run build

FROM nginx:alpine

COPY frontend/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/frontend/build /usr/share/nginx/html

EXPOSE 80

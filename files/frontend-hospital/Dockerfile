# syntax=docker/dockerfile:1

FROM node:16-alpine3.12 AS frontBuildHospital
WORKDIR /src
ARG FRONTEND_APP_SRC_URL=https://github.com/PSW-2021-ORG7/Managers-app/archive/refs/heads/master.tar.gz
SHELL ["/bin/ash", "-o", "pipefail", "-c"]
RUN apk --update --no-cache add \
    curl \
    tar \
    gettext && \
    curl -L "${FRONTEND_APP_SRC_URL}" | tar -xz && \
    mv "$(find . -maxdepth 1 -type d | tail -n 1)" app && \
    cd app/Frontend && \
    npm install
COPY environment.ts.template ./app
COPY ./shell-scripts/build-app.sh .
ARG HOSPITAL_API_URL
ARG INTEGRATION_API_URL
ARG PHARMACY_API_URL
RUN chmod +x ./build-app.sh && \
    ./build-app.sh ${HOSPITAL_API_URL} ${INTEGRATION_API_URL} ${PHARMACY_API_URL} 

FROM node:16-alpine3.12 AS frontBuildPharmacy
WORKDIR /src
ARG FRONTEND_APP_SRC_URL=https://github.com/PSW-2021-ORG7/Pharmacy/archive/refs/heads/master.tar.gz
SHELL ["/bin/ash", "-o", "pipefail", "-c"]
RUN apk --update --no-cache add \
    curl \
    tar \
    gettext && \
    curl -L "${FRONTEND_APP_SRC_URL}" | tar -xz && \
    mv "$(find . -maxdepth 1 -type d | tail -n 1)" app && \
    cd app/frontend && \
    npm install
COPY environment.ts.template ./app
COPY ./shell-scripts/build-app-ph.sh .
ARG HOSPITAL_API_URL
ARG INTEGRATION_API_URL
ARG PHARMACY_API_URL
RUN chmod +x ./build-app-ph.sh && \
    ./build-app-ph.sh ${HOSPITAL_API_URL} ${INTEGRATION_API_URL} ${PHARMACY_API_URL} 

FROM nginx:1.19.8-alpine AS gatewayWithFront
COPY --from=frontBuildHospital /app /usr/share/nginx/html/app/hospital
COPY --from=frontBuildPharmacy /app /usr/share/nginx/html/app/pharmacy
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./nginx/api_gateway.conf /etc/nginx/api_gateway.conf

FROM nginx:1.19.8-alpine AS gateway
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./nginx/api_gateway.conf /etc/nginx/api_gateway.conf
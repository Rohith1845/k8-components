FROM node:20.19.5-alpine3.22 AS build
WORKDIR /opt/server
COPY package.json .
COPY *.js .
RUN npm install

FROM node:20.19.5-alpine3.22
WORKDIR /opt/server
RUN apk update && \
    apk upgrade --no-cache
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop && \
    chown -R roboshop:roboshop /opt/server
EXPOSE 80
LABEL com.project="roboshop" \
      component="catalogue" \
      created_by="Rohith"
ENV MONGODB="true"\
    MONGO_URL="mongodb://mongodb:27017/catalogue"

COPY --from=build --chown=roboshop:roboshop /opt/server /opt/server/
USER roboshop
CMD [ "server.js" ]
ENTRYPOINT [ "node" ]

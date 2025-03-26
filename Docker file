FROM maven:3.9.6-eclipse-temurin-17-alpine AS builder

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src/ src/
RUN mvn clean package

FROM alpine:latest AS runner

WORKDIR /opt/
RUN apk add --no-cache curl tar openjdk17-jdk

ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk
ENV PATH="$JAVA_HOME/bin:$PATH"

ADD https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.102/bin/apache-tomcat-9.0.102.tar.gz /opt/
RUN tar -xzf apache-tomcat-9.0.102.tar.gz && rm apache-tomcat-9.0.102.tar.gz
RUN mv apache-tomcat-9.0.102 tomcat

WORKDIR /opt/tomcat/
RUN chmod -R +x /opt/tomcat

RUN nohup ./bin/startup.sh && sleep 5

RUN > conf/tomcat-users.xml
COPY tomcat-users.xml conf/tomcat-users.xml

COPY manager.xml conf/Catalina/localhost/manager.xml

RUN sed -i 's/8080/8086/g' conf/server.xml

COPY --from=builder /app/target/*.war webapps/ROOT.war

EXPOSE 8086

CMD ["sh", "-c", "./bin/catalina.sh stop || pkill -f 'catalina'"]
CMD ["sh", "-c", "./bin/catalina.sh run"]


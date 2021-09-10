FROM openjdk:11-jre-slim

COPY alfresco-ftr-distribution-7.0.0/ /opt/app/
RUN chmod +x /opt/app/file-transfer-receiver.jar

WORKDIR /opt/app
ENTRYPOINT ["java","-jar","file-transfer-receiver.jar"]
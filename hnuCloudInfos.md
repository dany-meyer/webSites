# hnuCloud Infos 194.95.24.122
- den MQTT Broker können Sie direkt benutzen (Port 1883)
- Um einen Grafana-User zu erhalten, wenden Sie sich bitte an Prof. D.Meyer
- um eine DB-Instanz zu erhalten wenden Sie sich bitte an Prof. D.Meyer per eMail; zur Nutzung ist eine Verbindung per VPN erforderlich

- um ein docker-Image zu deployen:
1. laden Sie das Image zunächst auf eine ContainerRegistry hoch z.B. gitHub Container Registry (ghcr)
   
2. verbinden Sie sich via VPN mit dem HNU Netzwerk

3. Login: http://194.95.24.122:8080
   Verfügbare Ports: 8081-8087 (für eure Container)
   Docker Befehle:
    - anlegen und wechseln in Eurer Unterverzeichnis (Befehle cd und mkdir)
    - Image herunterladen: z.B. docker pull ghcr.io/dany-meyer/my-spring-app:latest
    - Container starten: docker run -d -p 808x:XXXX --name mein-backend mein-image
       z.B. docker run -p 8081:10000 ghcr.io/dany-meyer/my-spring-app:latest
       (leitet port des docker image 10000 auf port 8081 um; App ist unter 194.95.24.122:8081 erreichbar)
    - Status prüfen: docker ps
    - Logs ansehen: docker logs meine-docker-id
    - Container stoppen/löschen: docker stop meine-docker-id
                                 docker rm meine-docker-id


### Weitere Ressourcen
- Docker Documentation: https://docs.docker.com/
- GitHub Container Registry: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry


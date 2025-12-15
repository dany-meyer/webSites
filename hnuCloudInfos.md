# hnuCloud Infos
- den MQTT Broker können Sie direkt benutzen

- um eine DB-Instanz zu erhalten wenden Sie sich bitte an Prof. D.Meyer per eMail; zur Nutzung ist eine Verbindung per VPN erforderlich

- um ein docker-Image zu deployen:
1. laden Sie das Image zunächst auf eine ContainerRegistry hoch z.B. gitHub Container Registry (ghcr)
2. Login
   Verfügbare Ports: 8081-8087 (für eure Container)
   Docker Befehle
    - Container starten: docker run -d -p 8081:8080 --name mein-backend mein-image
    - Status prüfen: docker ps
    - Logs ansehen: docker logs mein-backend
    - Container stoppen/löschen: docker stop mein-backend
                                 docker rm mein-backend


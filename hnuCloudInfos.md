# hnuCloud Infos
- den MQTT Broker können Sie direkt benutzen

- um eine DB-Instanz zu erhalten wenden Sie sich bitte an Prof. D.Meyer per eMail; zur Nutzung ist eine Verbindung per VPN erforderlich

- um ein docker-Image zu deployen:
1. laden Sie das Image zunächst auf eine ContainerRegistry hoch z.B. gitHub Container Registry (ghcr)
2. loggen Sie sich auf dem Rechner per ssh ein: ssh swe_deployer@194.95.24.122 (user=swe_deployer, pwd=deploy)
3. wechseln Sie in Ihr Verzeichnis
4. laden Sie Ihr docker image herunter und starten Sie es
Achten Sie darauf, dass nur die Ports 8080-8086 freigegeben sind, welchen Port Sie verwenden, erfragen Sie bitte bei Prof.D.Meyer


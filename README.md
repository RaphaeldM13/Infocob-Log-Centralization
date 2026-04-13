# Infocob

## SujetMise en place d'un système de récupération et d'exploitation de logs sur un réseau de machines Apaches, Linux et Windows## **Précisions**
- Outils open source
- Réseau d'une centaine de machines

##  Environnement

### **Outils:**
```
	Déploiement: Docker
	Exploitation: Grafana Loki
	Logs driver: Docker driver
	Storage logs: S3
	Logshipping côté node: Alloy
	node test: Apache
	Hyperviseur node test: Hyper-V
```

### **Config Hyper-V:**
```
	RAM: 32 GB
	Taille: 250 GB
	Réseau: Default Switch 
	ISO: Ubuntu server arm64 24.04.4
	démarrage sécurisé Certificat UEFI
	OpenSSH activé
	ip: 192.168.7.75
	
```

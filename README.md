# Scalable Web Application with ALB and Auto Scaling on AWS

**[English](#english) | [Français](#français)**

---

## English

### Overview

This project presents a highly available, auto-scaling web application 
architecture on AWS, designed as part of the AWS course capstone project 
(El Manara). It is built around a VPC spanning two Availability Zones, with 
public subnets hosting an Application Load Balancer (protected by AWS WAF) 
and NAT Gateways, and private subnets hosting an Auto Scaling Group of EC2 
instances and a Multi-AZ RDS database. Amazon CloudFront and Route 53 handle 
content delivery and DNS resolution, CloudWatch and SNS provide monitoring 
and alerting, and AWS Systems Manager enables secure, bastion-free 
administrative access.

### Architecture Diagram

![Architecture Diagram](diag.drawio.png.png)

**Flow summary:** Users resolve the domain via Route 53, then reach the 
application through CloudFront (cached content delivery) and AWS WAF 
(request filtering). Traffic then enters the VPC through the Internet 
Gateway to the Application Load Balancer, which distributes requests across 
EC2 instances managed by an Auto Scaling Group in private subnets. The 
application connects to a Multi-AZ RDS database, also isolated in private 
subnets. Outbound traffic from private instances (e.g. updates) goes through 
NAT Gateways. CloudWatch collects metrics from all key components and 
triggers SNS notifications to the administrator, who manages EC2 instances 
securely via AWS Systems Manager Session Manager (no bastion host, no 
exposed SSH port).

### Architecture Components

**Networking**
- **VPC** spanning 2 Availability Zones for high availability
- **Public subnets** (one per AZ): host the ALB and NAT Gateways
- **Private subnets** (one per AZ): host EC2 instances and RDS
- **Internet Gateway**: entry/exit point for the VPC to the internet
- **NAT Gateway** (one per AZ, with Elastic IP): allows private instances 
  outbound internet access without exposing them publicly

**Compute**
- **EC2 instances** running the application, deployed in private subnets
- **Auto Scaling Group**: automatically adjusts the number of EC2 instances 
  based on demand, and replaces unhealthy instances automatically
- **Launch Template**: defines the AMI, instance type, and security group 
  used when the ASG launches new instances

**Load Balancing & Content Delivery**
- **Application Load Balancer**: distributes incoming traffic across 
  healthy EC2 instances in both AZs
- **Amazon CloudFront**: caches static content at edge locations to reduce 
  latency and origin load

**Database**
- **Amazon RDS (Multi-AZ)**: primary instance with a synchronously 
  replicated standby in a different AZ, providing automatic failover

**DNS**
- **Route 53**: resolves the application domain name to CloudFront

**Security**
- **AWS WAF**: filters incoming requests against OWASP Top 10 threats, 
  attached to CloudFront
- **Security Groups**: enforce least-privilege access (ALB → EC2 → RDS only)
- **Private subnets**: EC2 and RDS have no direct exposure to the internet

**Access Management**
- **AWS Systems Manager (Session Manager)**: provides secure administrative 
  access to EC2 instances without a bastion host or open SSH port

**Monitoring**
- **Amazon CloudWatch**: collects metrics from ALB, ASG, and RDS
- **Amazon SNS**: sends alert notifications to the administrator when 
  CloudWatch alarms are triggered

### Security Considerations

- Compute and database layers are fully isolated from direct internet access
- All inbound traffic is filtered by WAF before reaching the application
- Administrative access uses Systems Manager instead of SSH/bastion hosts, 
  removing the need for any inbound management port
- Security Groups follow the principle of least privilege: each tier only 
  accepts traffic from the tier directly in front of it

### Deployment

This repository documents the architecture design and diagram. Deployment 
would follow this order: VPC and subnets → Internet/NAT Gateways → RDS → 
ALB and Auto Scaling Group → CloudFront and WAF → Route 53 → CloudWatch 
alarms and SNS topic.

### Future Work

- Infrastructure as Code implementation (Terraform)
- Full deployment on AWS with a live demo

---

## Français

### Aperçu

Ce projet présente une architecture d'application web hautement disponible 
et auto-scalable sur AWS, réalisée dans le cadre du projet de fin de cours 
AWS (El Manara). Elle repose sur un VPC réparti sur deux zones de 
disponibilité, avec des subnets publics hébergeant un Application Load 
Balancer (protégé par AWS WAF) et des NAT Gateways, et des subnets privés 
hébergeant un Auto Scaling Group d'instances EC2 ainsi qu'une base de 
données RDS Multi-AZ. Amazon CloudFront et Route 53 gèrent la distribution 
de contenu et la résolution DNS, CloudWatch et SNS assurent la surveillance 
et les alertes, et AWS Systems Manager permet un accès administratif 
sécurisé sans bastion.

### Diagramme d'architecture

![Diagramme d'architecture](architecture-diagram.png)

**Résumé du flux :** Les utilisateurs résolvent le domaine via Route 53, 
puis accèdent à l'application via CloudFront (distribution de contenu mis 
en cache) et AWS WAF (filtrage des requêtes). Le trafic entre ensuite dans 
le VPC via l'Internet Gateway jusqu'à l'Application Load Balancer, qui 
répartit les requêtes entre les instances EC2 gérées par un Auto Scaling 
Group dans les subnets privés. L'application se connecte à une base RDS 
Multi-AZ, elle aussi isolée dans les subnets privés. Le trafic sortant des 
instances privées (mises à jour, etc.) passe par les NAT Gateways. 
CloudWatch collecte les métriques de tous les composants clés et déclenche 
des notifications SNS vers l'administrateur, qui gère les instances EC2 de 
façon sécurisée via AWS Systems Manager Session Manager (sans bastion, sans 
port SSH exposé).

### Composants de l'architecture

**Réseau**
- **VPC** réparti sur 2 zones de disponibilité pour la haute disponibilité
- **Subnets publics** (un par AZ) : hébergent l'ALB et les NAT Gateways
- **Subnets privés** (un par AZ) : hébergent les instances EC2 et RDS
- **Internet Gateway** : point d'entrée/sortie du VPC vers internet
- **NAT Gateway** (un par AZ, avec Elastic IP) : permet aux instances 
  privées de sortir vers internet sans être exposées publiquement

**Calcul**
- **Instances EC2** exécutant l'application, déployées en subnets privés
- **Auto Scaling Group** : ajuste automatiquement le nombre d'instances EC2 
  selon la charge, et remplace automatiquement les instances défaillantes
- **Launch Template** : définit l'AMI, le type d'instance et le security 
  group utilisés lors du lancement de nouvelles instances par l'ASG

**Répartition de charge & distribution de contenu**
- **Application Load Balancer** : répartit le trafic entrant entre les 
  instances EC2 saines des deux AZ
- **Amazon CloudFront** : met en cache le contenu statique sur les edge 
  locations pour réduire la latence

**Base de données**
- **Amazon RDS (Multi-AZ)** : instance primaire avec un standby répliqué de 
  façon synchrone dans une autre AZ, assurant un failover automatique

**DNS**
- **Route 53** : résout le nom de domaine de l'application vers CloudFront

**Sécurité**
- **AWS WAF** : filtre les requêtes entrantes contre les menaces OWASP 
  Top 10, attaché à CloudFront
- **Security Groups** : appliquent le principe du moindre privilège 
  (ALB → EC2 → RDS uniquement)
- **Subnets privés** : EC2 et RDS ne sont jamais exposés directement à 
  internet

**Gestion des accès**
- **AWS Systems Manager (Session Manager)** : fournit un accès 
  administratif sécurisé aux instances EC2 sans bastion ni port SSH ouvert

**Supervision**
- **Amazon CloudWatch** : collecte les métriques de l'ALB, de l'ASG et de 
  RDS
- **Amazon SNS** : envoie des notifications d'alerte à l'administrateur 
  lorsqu'une alarme CloudWatch se déclenche

### Considérations de sécurité

- Les couches calcul et base de données sont totalement isolées de tout 
  accès direct depuis internet
- Tout le trafic entrant est filtré par le WAF avant d'atteindre 
  l'application
- L'accès administratif utilise Systems Manager plutôt que SSH/bastion, 
  supprimant le besoin d'un port de gestion entrant
- Les Security Groups suivent le principe du moindre privilège : chaque 
  niveau n'accepte le trafic que du niveau qui le précède directement

### Déploiement

Ce dépôt documente la conception de l'architecture et son diagramme. Le 
déploiement suivrait cet ordre : VPC et subnets → Internet/NAT Gateways → 
RDS → ALB et Auto Scaling Group → CloudFront et WAF → Route 53 → alarmes 
CloudWatch et topic SNS.

### Pistes d'évolution

- Implémentation en Infrastructure as Code (Terraform)
- Déploiement complet sur AWS avec démonstration live

---

## License

This project is licensed under the MIT License.
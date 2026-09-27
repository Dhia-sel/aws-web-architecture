# Security Considerations

**[English](#english) | [Français](#français)**

---

## English

### Network Isolation
- EC2 instances and the RDS database are deployed exclusively in private 
  subnets, with no public IP addresses and no direct route to the internet.
- The only publicly reachable component is the Application Load Balancer, 
  itself shielded by AWS WAF and CloudFront.

### Perimeter Filtering
- AWS WAF is attached to CloudFront and filters incoming requests against 
  OWASP Top 10 threats before they reach the application.

### Least-Privilege Access Control
- Security Groups are scoped per tier: the ALB Security Group accepts 
  traffic from the internet on HTTP/HTTPS only; the EC2 Security Group 
  accepts traffic only from the ALB Security Group; the RDS Security Group 
  accepts traffic only from the EC2 Security Group on the database port.
- No Security Group allows unrestricted inbound access (0.0.0.0/0) except 
  the ALB on ports 80/443.

### Administrative Access
- AWS Systems Manager Session Manager is used for administrative access to 
  EC2 instances, removing the need for a bastion host or any open SSH port.
- This also removes the need to manage or rotate SSH key pairs.

### Data Layer Protection
- RDS is deployed in Multi-AZ mode for automatic failover and is only 
  reachable from the application tier — never directly from the internet 
  or from administrative access paths.

### Monitoring & Alerting
- CloudWatch collects metrics from the ALB, Auto Scaling Group, and RDS.
- SNS notifies the administrator when abnormal activity or thresholds 
  trigger a CloudWatch alarm, enabling timely response.

### Threats Considered (High Level)
| Threat | Mitigation |
|---|---|
| Application-layer attacks (SQLi, XSS, etc.) | AWS WAF (OWASP Top 10 rules) |
| Direct access to compute/database | Private subnets, no public IPs |
| Unauthorized administrative access | Systems Manager instead of SSH/bastion |
| Single point of failure | Multi-AZ deployment (ALB, ASG, RDS) |
| Unnoticed anomalies | CloudWatch alarms + SNS notifications |

---

## Français

### Isolation réseau
- Les instances EC2 et la base RDS sont déployées exclusivement dans des 
  subnets privés, sans adresse IP publique ni route directe vers internet.
- Le seul composant accessible publiquement est l'Application Load 
  Balancer, lui-même protégé par AWS WAF et CloudFront.

### Filtrage en périphérie
- AWS WAF est attaché à CloudFront et filtre les requêtes entrantes contre 
  les menaces OWASP Top 10 avant qu'elles n'atteignent l'application.

### Contrôle d'accès au moindre privilège
- Les Security Groups sont définis par niveau : le Security Group de l'ALB 
  accepte le trafic internet en HTTP/HTTPS uniquement ; celui des EC2 
  n'accepte le trafic que du Security Group de l'ALB ; celui de RDS 
  n'accepte le trafic que du Security Group des EC2 sur le port de la base.
- Aucun Security Group n'autorise un accès entrant illimité (0.0.0.0/0), 
  sauf l'ALB sur les ports 80/443.

### Accès administratif
- AWS Systems Manager Session Manager est utilisé pour l'accès 
  administratif aux instances EC2, supprimant le besoin d'un bastion ou de 
  tout port SSH ouvert.
- Cela supprime aussi le besoin de gérer ou faire tourner des paires de 
  clés SSH.

### Protection de la couche de données
- RDS est déployée en mode Multi-AZ pour un failover automatique, et n'est 
  accessible que depuis la couche applicative — jamais directement depuis 
  internet ni depuis les chemins d'accès administratifs.

### Surveillance et alertes
- CloudWatch collecte les métriques de l'ALB, de l'Auto Scaling Group et 
  de RDS.
- SNS notifie l'administrateur lorsqu'une activité anormale ou un seuil 
  déclenche une alarme CloudWatch, permettant une réaction rapide.

### Menaces considérées (vue d'ensemble)
| Menace | Mitigation |
|---|---|
| Attaques applicatives (SQLi, XSS, etc.) | AWS WAF (règles OWASP Top 10) |
| Accès direct au calcul/à la base | Subnets privés, pas d'IP publique |
| Accès administratif non autorisé | Systems Manager plutôt que SSH/bastion |
| Point de défaillance unique | Déploiement Multi-AZ (ALB, ASG, RDS) |
| Anomalies non détectées | Alarmes CloudWatch + notifications SNS |
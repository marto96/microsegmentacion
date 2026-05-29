# Matriz de Cumplimiento - NetworkPolicies (Namespace: novaship)

| ID   | Origen           | Destino         | Puerto | Protocolo | Estado     | NetworkPolicy / Justificación                                      |
|------|-----------------|----------------|--------|-----------|-----------|--------------------------------------------------------------------|
| F01  | client           | portal-web     | 80     | TCP       | ✅ Permitido | allow-client-to-portal-web-ingress / allow-client-egress-to-portal-web |
| F02  | portal-web       | orders-api     | 8080   | TCP       | ✅ Permitido | allow-portal-web-to-orders-api-ingress / allow-portal-web-egress-to-orders-api |
| F03  | orders-api       | inventory-api  | 8081   | TCP       | ✅ Permitido | allow-orders-api-to-inventory-api-ingress / allow-orders-api-egress-to-inventory-api |
| F04  | orders-api       | dispatch-api   | 8082   | TCP       | ✅ Permitido | allow-orders-api-to-dispatch-api-ingress / allow-orders-api-egress-to-dispatch-api |
| F05  | inventory-api    | database       | 5432   | TCP       | ✅ Permitido | allow-inventory-api-to-database-ingress / allow-inventory-api-egress-to-database |
| F06  | dispatch-api     | database       | 5432   | TCP       | ✅ Permitido | allow-dispatch-api-to-database-ingress / allow-dispatch-api-egress-to-database |
| F07  | Pods autorizados | CoreDNS        | 53     | UDP/TCP   | ✅ Permitido | allow-dns-egress                                                   |
| B01  | portal-web       | database       | 5432   | TCP       | ❌ Bloqueado | No hay política que permita este flujo                             |
| B02  | portal-web       | inventory-api  | 8081   | TCP       | ❌ Bloqueado | Solo orders-api puede acceder a inventory-api                     |
| B03  | portal-web       | dispatch-api   | 8082   | TCP       | ❌ Bloqueado | Solo orders-api puede acceder a dispatch-api                      |
| B04  | orders-api       | database       | 5432   | TCP       | ❌ Bloqueado | Solo inventory-api y dispatch-api pueden acceder a database       |
| B05  | attacker         | orders-api     | 8080   | TCP       | ❌ Bloqueado | No hay política que permita acceso desde attacker                 |
| B06  | attacker         | inventory-api  | 8081   | TCP       | ❌ Bloqueado | No hay política que permita acceso desde attacker                 |
| B07  | attacker         | dispatch-api   | 8082   | TCP       | ❌ Bloqueado | No hay política que permita acceso desde attacker                 |
| B08  | attacker         | database       | 5432   | TCP       | ❌ Bloqueado | No hay política que permita acceso desde attacker                 |

---

## Controles Base Implementados

| Control                               | NetworkPolicy                     | Cumple |
|---------------------------------------|----------------------------------|--------|
| Default Deny Ingress                   | deny-all-ingress                 | ✅     |
| Default Deny Egress                    | deny-all-egress                  | ✅     |
| Resolución DNS autorizada              | allow-dns-egress                 | ✅     |
| Segmentación Web → API                 | Policies F01-F02                 | ✅     |
| Segmentación API → Servicios Internos  | Policies F03-F04                 | ✅     |
| Segmentación Servicios → Base de Datos | Policies F05-F06                 | ✅     |
| Prevención de movimiento lateral       | Default Deny + reglas explícitas | ✅     |
| Aislamiento del pod attacker           | Default Deny + ausencia de permisos | ✅     |

---

## Evidencias de Validación Sugeridas

| ID      | Prueba                                  | Resultado esperado           |
|---------|----------------------------------------|-----------------------------|
| F01     | `curl portal-web:80` desde client       | Éxito                       |
| F02     | `curl orders-api:8080` desde portal-web | Éxito                       |
| F03     | `curl inventory-api:8081` desde orders-api | Éxito                   |
| F04     | `curl dispatch-api:8082` desde orders-api | Éxito                     |
| F05     | `nc -zv database 5432` desde inventory-api | Éxito                    |
| F06     | `nc -zv database 5432` desde dispatch-api | Éxito                     |
| B01-B08 | curl/nc desde origen indicado           | Timeout o Connection refused |
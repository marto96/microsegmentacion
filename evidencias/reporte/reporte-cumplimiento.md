# Reporte de cumplimiento - Microsegmentación en Kubernetes

## 1. Escenario

La aplicación NovaShip Logistics se encuentra desplegada en Kubernetes dentro del namespace `novaship`, con una arquitectura basada en microservicios compuesta por los siguientes componentes:

- client
- portal-web
- orders-api
- inventory-api
- dispatch-api
- database
- attacker
- CoreDNS

Se implementaron NetworkPolicies siguiendo un modelo **Default Deny (deny all)** para restringir el tráfico entre componentes y permitir únicamente las comunicaciones necesarias para el funcionamiento de la plataforma.

El objetivo principal fue prevenir movimientos laterales dentro del clúster y garantizar que cada servicio únicamente pueda comunicarse con los componentes estrictamente necesarios para cumplir su función.

---

## 2. Evidencia recolectada

Se almacenaron evidencias de:

- Estado de Pods desplegados en el namespace `novaship`
- Services y Endpoints asociados a cada componente
- Labels utilizadas por los selectores de las NetworkPolicies
- NetworkPolicies aplicadas en el namespace
- Verificación de políticas de aislamiento (deny-all-ingress y deny-all-egress)
- Evidencias de resolución DNS interna mediante CoreDNS
- Pruebas positivas de conectividad autorizada
- Pruebas negativas de conectividad no autorizada
- Validación de aislamiento del pod attacker

---

## 3. Resultados principales

### Flujos permitidos confirmados

Se validó correctamente la comunicación entre los componentes autorizados:

| ID | Flujo |
|----|--------|
| F01 | client → portal-web (TCP/80) |
| F02 | portal-web → orders-api (TCP/8080) |
| F03 | orders-api → inventory-api (TCP/8081) |
| F04 | orders-api → dispatch-api (TCP/8082) |
| F05 | inventory-api → database (TCP/5432) |
| F06 | dispatch-api → database (TCP/5432) |
| F07 | Pods autorizados → CoreDNS (UDP/TCP 53) |

Las pruebas de conectividad demostraron que los servicios mantienen su funcionamiento normal y que la aplicación puede procesar solicitudes siguiendo el flujo de negocio definido.

### Flujos bloqueados confirmados

Se verificó el bloqueo efectivo de las comunicaciones no autorizadas:

| ID | Flujo |
|----|--------|
| B01 | portal-web → database (TCP/5432) |
| B02 | portal-web → inventory-api (TCP/8081) |
| B03 | portal-web → dispatch-api (TCP/8082) |
| B04 | orders-api → database (TCP/5432) |
| B05 | attacker → orders-api (TCP/8080) |
| B06 | attacker → inventory-api (TCP/8081) |
| B07 | attacker → dispatch-api (TCP/8082) |
| B08 | attacker → database (TCP/5432) |

Las pruebas negativas evidenciaron que las conexiones no autorizadas fueron rechazadas o finalizaron por timeout, confirmando la efectividad de las políticas de microsegmentación.

### Controles de seguridad implementados

- Política global de denegación de tráfico entrante (`deny-all-ingress`)
- Política global de denegación de tráfico saliente (`deny-all-egress`)
- Permisos explícitos únicamente para flujos de negocio autorizados
- Permisos específicos para resolución DNS mediante CoreDNS
- Aislamiento completo del pod `attacker`
- Prevención de acceso directo a la base de datos desde capas superiores
- Prevención de bypass de servicios intermedios
- Reducción de la superficie de ataque y del riesgo de movimiento lateral

---

## 4. Conclusión

Las pruebas realizadas demuestran que la estrategia de microsegmentación implementada en el namespace `novaship` cumple con los requisitos de seguridad definidos para la plataforma NovaShip Logistics.

Las NetworkPolicies permiten exclusivamente los flujos necesarios para la operación de la aplicación, manteniendo la comunicación entre los servicios autorizados y garantizando la resolución DNS interna.

Asimismo, se confirmó el bloqueo efectivo de accesos no autorizados, incluyendo intentos de conexión desde el pod `attacker`, accesos directos a la base de datos y comunicaciones que omiten los servicios intermedios definidos por la arquitectura.

Como resultado, la solución implementada reduce significativamente el riesgo de movimiento lateral dentro del clúster Kubernetes y fortalece el modelo de seguridad basado en el principio de mínimo privilegio.
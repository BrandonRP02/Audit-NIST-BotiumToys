# Auditoría de Seguridad Interna — Botium Toys

> **Autor:** Brandon Rondon
> **Fecha:** Junio 2026  
> **Alcance:** Evaluación de controles de seguridad y cumplimiento normativo  
> **Nivel de riesgo general:** 8/10

---

## Contexto

Botium Toys es una empresa de comercio electrónico que ha crecido bastante rápido en los últimos años. El problema es que ese crecimiento no vino acompañado de una maduración proporcional en su postura de seguridad. Me pidieron hacer una revisión integral de sus controles técnicos, administrativos y físicos, además de evaluar dónde están parados frente a PCI DSS, GDPR y SOC.

Lo que encontré no es bonito, pero tampoco es raro para una empresa de este tamaño que nunca tuvo un equipo de seguridad dedicado.

---

## 1. Evaluación de Controles

| Control | ¿Implementado? | Observaciones |
| :--- | :---: | :--- |
| Privilegio Mínimo |  No | Todos los empleados tienen acceso a todo. Datos financieros, PII de clientes... sin ningún tipo de restricción. Este es probablemente el hallazgo más crítico. |
| DRP (Recuperación ante Desastres) |  No | No existe plan de recuperación. Si mañana se cae el servidor principal, no hay un procedimiento definido para levantar operaciones. |
| Políticas de Contraseña | Parcial | Existen políticas escritas, pero son muy básicas. No exigen complejidad real ni rotación periódica. Están ahí más por formalidad que por protección. |
| Separación de Funciones |  No | No hay segregación de roles. La misma persona que administra la base de datos podría aprobar transacciones. Riesgo alto de fraude interno. |
| Firewall |  Sí | Funciona bien. Tiene reglas configuradas y el equipo de TI lo revisa regularmente. Es de los pocos controles que realmente están operativos. |
| IDS |  No | No tienen ningún sistema de detección de intrusos, ni en red ni en endpoints. Básicamente si alguien entra, nadie se entera hasta que es demasiado tarde. |
| Backups | No | No hacen respaldos. Esto es grave. Cualquier incidente de ransomware o fallo de hardware significaría pérdida total de datos. |
| Antivirus |  Sí | Tienen antivirus desplegado en los endpoints y lo mantienen actualizado. Bien en este punto. |
| Monitoreo de Sistemas Legados |  Parcial | Hay supervisión, pero es manual y no sigue un calendario. Depende de que alguien se acuerde de revisarlo. No hay alertas automáticas. |
| Cifrado |  No | Datos bancarios y de tarjetas viajan en texto plano internamente. Esto solo ya es una violación directa de PCI DSS. |
| Gestión de Contraseñas |  No | No usan ningún gestor centralizado. Cada quien maneja sus credenciales como puede, lo que genera tickets de soporte constantemente y contraseñas débiles. |
| Cerraduras Físicas |  Sí | Las oficinas, tienda y almacén tienen cerraduras adecuadas. Sin quejas aquí. |
| CCTV | Sí | Sistema de cámaras operativo y con buena cobertura del perímetro. |
| Detección de Incendios | Sí | Detectores de humo y sistema de supresión funcionando. Cumple con normativa. |

---

## 2. Cumplimiento Normativo

### PCI DSS

Este es donde más duele. Si procesan pagos con tarjeta (y lo hacen), prácticamente no cumplen con nada.

| Requisito | ¿Cumple? | Detalle |
| :--- | :---: | :--- |
| Acceso restringido a datos de tarjetas |  No | Cualquier empleado puede ver información de tarjetas de crédito. No hay controles de acceso diferenciados. |
| Entornos seguros para procesamiento |  No | La base de datos no está segmentada. Todo corre en la misma red sin aislamiento. |
| Cifrado de datos bancarios | No | Ni en reposo ni en tránsito. Los datos de pago se mueven en texto plano dentro de la red. |
| Gestión robusta de credenciales | No | Políticas débiles + sin gestor centralizado = riesgo alto. |

### GDPR

Relevante porque venden a clientes de la UE.

| Requisito | ¿Cumple? | Detalle |
| :--- | :---: | :--- |
| Protección de datos de usuarios UE | No | El acceso abierto a datos y la falta de cifrado van en contra del principio de privacidad por diseño. |
| Notificación de brechas (<72h) |  Sí | Tienen un plan de respuesta a incidentes aprobado. Al menos esto está cubierto. |
| Inventario de activos de datos |  No | No tienen un catálogo formal de qué datos manejan, dónde se almacenan ni cómo fluyen. Necesitan armar esto cuanto antes. |
| Políticas de privacidad documentadas |  Sí | Las tienen y se comunican al personal. Ok en este frente. |

### SOC (Type 1 / Type 2)

| Requisito | ¿Cumple? | Detalle |
| :--- | :---: | :--- |
| Políticas formales de acceso |  No | Sin un modelo RBAC, las políticas de acceso son solo papel. No se aplican en la práctica. |
| Confidencialidad de PII/SPII |  No | Hay brechas en los controles lógicos. Datos personales de clientes quedan expuestos. |
| Integridad de datos |  Sí | El equipo de TI tiene controles para validar consistencia en la base de datos. Funciona razonablemente bien. |
| Disponibilidad |  Sí | Los servicios se mantienen operativos. No han tenido downtime significativo... aunque sin backups, la suerte no va a durar para siempre. |

---

## 3. Recomendaciones

El riesgo actual es alto (8/10). Estas son las acciones que priorizaría, en orden de urgencia:

**1. Implementar RBAC ya.**  
Lo primero es sacar a todo el mundo del acceso universal. Definir roles, asignar permisos mínimos según la función de cada persona. Esto solo ya baja el riesgo de varios hallazgos de golpe: privilegio mínimo, separación de funciones, PCI DSS y SOC.

**2. Cifrar todo.**  
AES-256 para datos en reposo, TLS 1.3 para tránsito. No hay excusa para que datos de tarjetas viajen en texto plano en 2026. Esto es obligatorio para PCI DSS y GDPR.

**3. Centralizar la gestión de identidades.**  
Necesitan un IAM. Contraseñas complejas, rotación obligatoria y MFA en todas las cuentas corporativas. El costo es mínimo comparado con el riesgo de una brecha por credenciales comprometidas.

**4. Armar un DRP y hacer backups.**  
Plan de recuperación documentado + backups automatizados e inmutables, almacenados fuera de la red principal. Si un ransomware les pega hoy, pierden todo. Este punto debería quitarle el sueño a alguien.

**5. Instalar un IDS.**  
El firewall solo no alcanza. Necesitan detección de intrusos para identificar tráfico anómalo que pase las reglas del perímetro. Un NIDS en los segmentos críticos como mínimo.

---

*Nota: Este informe se elaboró como parte de un ejercicio práctico de auditoría de seguridad. Los hallazgos reflejan un escenario simulado basado en la documentación proporcionada por la organización.*

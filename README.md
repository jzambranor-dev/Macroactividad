# MediSecure — Portal de Resultados Médicos

Caso de estudio de ciberseguridad que implementa un **módulo de autenticación frontend con defensa en profundidad** y un **plan de respuesta a incidentes basado en NIST SP 800-61 Rev. 2**, aplicado a un escenario de ataque de fuerza bruta contra una plataforma de resultados médicos.

## Escenario de Amenaza

Un atacante ejecuta técnicas de **OSINT** para recopilar información del administrador de MediSecure (`admin@medisecure.com`) y posteriormente lanza un **ataque de fuerza bruta** que compromete la cuenta administrativa. Esto representa un riesgo crítico de exfiltración de datos protegidos bajo **HIPAA** (PHI — Protected Health Information).

- **Nivel de severidad:** Crítico
- **IP del atacante:** 203.0.113.42
- **Vector:** OSINT + Fuerza bruta
- **Impacto:** Acceso no autorizado a resultados médicos, posible modificación de registros, violación HIPAA

## Estructura del Proyecto

```
Macroactividad/
├── login.html    # Módulo de autenticación con defensa en profundidad
├── index.html    # Portal/Dashboard con panel de respuesta a incidentes
├── docs/
│   └── Plan de Respuesta a Incidentes.docx
└── README.md
```

## Arquitectura de Seguridad — login.html

El módulo de autenticación implementa 5 capas de defensa:

| Capa | Nombre | Descripción |
|------|--------|-------------|
| 0 | HTML Defensivo | Atributos `autocomplete="off"`, `spellcheck="false"`, `maxlength`, `required` |
| 1 | Anti-Fuerza Bruta | Límite de 3 intentos fallidos, bloqueo total de la interfaz |
| 2 | Higiene de Datos | Sanitización por allowlist, detección de patrones maliciosos (SQLi, XSS, path traversal, inyección de comandos) |
| 3 | Política de Clave | Mínimo 8 caracteres, mayúscula, minúscula y número obligatorios |
| 4 | Autenticación Opaca | Mensajes genéricos que no revelan qué campo falló (previene enumeración de usuarios) |
| T | Mini-SIEM | Logging estructurado en consola con timestamp ISO 8601 y niveles de severidad |

## Plan de Respuesta a Incidentes — index.html

El dashboard implementa las 4 fases del estándar **NIST SP 800-61 Rev. 2** como acciones interactivas:

### Fase 1: Preparación (Defensa Previa)
- Políticas de contraseñas robustas
- Monitoreo continuo SIEM
- Backups en GitHub con verificación de integridad SHA-256
- Verificación de sesión (`sessionStorage`)

### Fase 2: Detección y Análisis
- Log de actividad con IoCs (Indicadores de Compromiso)
- Detección de fuerza bruta (47 intentos en 12 minutos)
- Acceso fuera de horario laboral
- Enumeración de usuarios / patrones OSINT

### Fase 3: Contención, Erradicación y Recuperación
- **3A Contención:** Modo mantenimiento, bloqueo de IP, desactivación de cuenta
- **3B Erradicación:** Limpieza de `localStorage`/tokens, revisión de persistencia
- **3C Recuperación:** Restablecimiento de credenciales, restauración desde backup, rate limiting

### Fase 4: Post-Incidente (Lecciones Aprendidas)
- Causa raíz: ausencia de MFA (ref: OWASP 2021)
- Acción correctiva 1: MFA obligatorio para cuentas privilegiadas
- Acción correctiva 2: CAPTCHA adaptativo en flujo de autenticación
- Auditoría trimestral de permisos de acceso

## Comité de Crisis

| Rol | Nombre | Responsabilidad |
|-----|--------|-----------------|
| Comandante del Incidente | Carlos Martínez (CISO) | Decisión de desconectar el portal |
| Líder Técnico | — | Análisis forense, parches CI/CD, restauración |
| Comunicaciones | Elena Rojas (RRPP) | Notificación a pacientes y entes reguladores (HIPAA) |

## Demostración

### Credenciales de prueba
- **Usuario:** `admin_seguro`
- **Contraseña:** `ClaveSegura1`

### Flujo de demostración

1. Abrir `login.html` en el navegador
2. Abrir la consola del navegador (`F12 > Console`) para ver los logs SIEM
3. Probar escenarios de ataque:
   - Inyección SQL: escribir `' OR 1=1 --` como usuario
   - Fuerza bruta: fallar 3 veces para activar el bloqueo
   - Contraseña débil: escribir `abc` como contraseña
4. Autenticarse con las credenciales correctas para acceder al portal
5. En el portal (`index.html`), ejecutar las fases en orden:
   - **Contención** (amarillo) — activa el modo mantenimiento
   - **Erradicación** (rojo) — limpia tokens y sesiones comprometidas
   - **Recuperación** (verde) — restaura el sistema a estado operativo
   - **Post-Incidente** (púrpura) — muestra el informe con lecciones aprendidas

## Tecnologías

- HTML5 semántico con atributos de seguridad
- CSS3 (diseño responsive, glassmorphism, gradientes)
- JavaScript vanilla (sin dependencias externas)
- Google Fonts (Inter)

## Referencias

- NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide
- OWASP Top 10 (2021)
- HIPAA Security Rule — 45 CFR Part 164

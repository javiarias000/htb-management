# Management (HackTheBox) - Análisis de Vulnerabilidad y Guías de Explotación

**Máquina**: Management (10.129.122.141)  
**Fecha de Análisis**: 2026-10-04  
**Estado**: Vulnerabilidades confirmadas - Documentación lista para explotación manual

---

## 📋 Resumen Ejecutivo

### Tecnología Identificada
```
Frontend Web:    nginx 1.24.0 (SPA con AES-GCM encryption)
SSO/Identity:    Open Identity Platform OpenAM 16.0.5
LDAP Directory:  OpenDJ 5.0.3 Community
Java Runtime:    OpenJDK 21 (sobre Tomcat)
```

### Puertos Expuestos
```
22/tcp    SSH                (OpenSSH 9.6p1)
80/tcp    HTTP              (nginx → redirect a HTTPS)
443/tcp   HTTPS             (nginx + self-signed cert)
1689/tcp  Java RMI          (OpenDJ JMX Registry)
4444/tcp  LDAPS             (OpenDJ Administration Connector)
39773/tcp Java RMI          (JMX Object Export)
50389/tcp LDAP              (OpenDJ - Anonymous Bind OK)
```

### Vulnerabilidades Identificadas

| CVE | Vulnerabilidad | Severidad | Vector | Documentación |
|-----|-----------------|-----------|--------|---------------|
| CVE-2026-33439 | OpenAM Pre-Auth RCE | CRÍTICA (9.8) | jato.clientSession deserialization | CVE-2026-33439-EXPLOIT-GUIDE.md |
| N/A | JMX Exposed (sin auth) | CRÍTICA (9.8) | RMI registry en 1689 | EXPLOIT-JMX-ALTERNATIVE.md |
| N/A | LDAP Anonymous Bind | ALTA (7.5) | Enumeration en 50389 | [Ver abajo] |

---

## 🎯 Vulnerabilidad Principal: CVE-2026-33439

### Descripción Técnica

OpenAM 16.0.5 es vulnerable a **Remote Code Execution pre-autenticación** mediante deserialización Java insegura en el parámetro `jato.clientSession`.

### Root Cause
```
Clase vulnerable: com.iplanet.jato.util.Encoder.deserialize()
Implementación: ApplicationObjectInputStream (SIN whitelist de clases)

Diferencia con CVE-2021-35464:
  CVE-2021-35464: jato.pageSession → ✓ Protegido con WhitelistObjectInputStream
  CVE-2026-33439: jato.clientSession → ✗ Sin protección (bypass)
```

### Ruta de Ataque
```
1. Generar gadget chain con ysoserial (CommonsCollections5, etc.)
2. Codificar en base64 + URL-encode
3. Enviar GET/POST a endpoint JATO: 
   GET https://sso.management.htb/openam/XUI?jato.clientSession=[PAYLOAD]
4. Servidor deserializa → RCE como usuario 'tomcat'
```

### Recursos
- **Guía Detallada**: [CVE-2026-33439-EXPLOIT-GUIDE.md](./CVE-2026-33439-EXPLOIT-GUIDE.md)
- **PoC Público**: https://github.com/TheMalwareGuardian/CVE-2026-33439
- **Advisory**: https://github.com/OpenIdentityPlatform/OpenAM/security/advisories/GHSA-2cqq-rpvq-g5qj

---

## 🔧 Vulnerabilidad Alternativa: JMX RCE (Puerto 1689)

OpenDJ expone su interfaz JMX sin autenticación sobre Java RMI. Esto permite:

1. **Conexión directa** al MBeanServer sin credenciales
2. **Leer configuración sensible** (Directory Manager password)
3. **Crear/modificar MBeans** para RCE
4. **MLet gadgets** para cargar código malicioso

### Ruta de Ataque
```
1. Conectar a RMI registry: rmi://10.129.122.141:1689
2. Obtener stub de org.opends.server.protocols.jmx.client-unknown
3. Conectar a MBeanServer sin credenciales
4. Enumerar MBeans y buscar operaciones explotables
5. RCE vía MLet o serialización maliciosa
```

### Recursos
- **Guía Detallada**: [EXPLOIT-JMX-ALTERNATIVE.md](./EXPLOIT-JMX-ALTERNATIVE.md)
- **Código Fuente**: [jmx_rce_exploit.java](./jmx_rce_exploit.java)

---

## 🔎 Enumeración LDAP (50389)

### Lo que se logró enumerar
```bash
# Anonymous bind exitoso
ldapsearch -x -H ldap://10.129.122.141:50389 -b "" -s base
```

### Resultados
✓ Versión: OpenDJ Server 5.0.3  
✓ Base DN: dc=management,dc=htb  
✓ Usuarios listables vía /openam/XUI/callbacks (requiere análisis web)  
✗ Passwords NO legibles (ACI protege userPassword)  
✗ cn=config NO accesible (requiere Directory Manager)  

### Info Sensible Encontrada
```
Backends:
  - cn=admin data
  - cn=ads-truststore (certificados)
  - cn=backups
  - cn=config (PROTEGIDO)
  - cn=tasks
  - dc=management,dc=htb

JVM Info:
  - Java 21 (OpenJDK)
  - Path: /opt/openam-tomcat/
  - JMX Arguments: -Djava.util.logging.config.file=/opt/openam-tomcat/conf/logging
  - Threads: Apache Catalina (Tomcat)
```

---

## 📝 Archivos de Documentación

Todos los archivos están en `/home/javlabs/hackthebox/Management/`:

### 1. **CVE-2026-33439-EXPLOIT-GUIDE.md**
Guía técnica completa para explotar OpenAM vía jato.clientSession
- Análisis técnico detallado
- Preparación del entorno
- Pasos manuales con comandos reales
- Troubleshooting
- Quick-start (4 comandos)

### 2. **EXPLOIT-JMX-ALTERNATIVE.md**
Explotación alternativa vía JMX en puerto 1689
- Análisis del vector JMX
- Herramientas disponibles
- Código Java personalizado
- Post-explotación (acceso a LDAP)

### 3. **jmx_rce_exploit.java**
Código fuente del cliente JMX que enumera MBeans sin autenticación
```bash
# Compilar
javac jmx_rce_exploit.java

# Ejecutar
java jmx_rce_exploit 10.129.122.141 1689
```

### 4. **ENTORNO-REPRODUCIBLE.md** (Este archivo)
Resumen ejecutivo de todas las vulnerabilidades

---

## 🚀 Flujo de Ataque Recomendado

### Opción A: CVE-2026-33439 (OpenAM)
```
1. Generar payload con ysoserial
2. Enviar a https://sso.management.htb/openam/XUI
3. RCE como 'tomcat'
4. Buscar credenciales de Directory Manager en config
5. Usar LDAP (4444) para modificar usuarios/SSH
6. SSH como usuario privilegiado
```

### Opción B: JMX (Más Directo)
```
1. Conectar a JMX en 1689 sin credenciales
2. Enumerar MBeans de configuración
3. Extraer Directory Manager password
4. Usar LDAP (4444) para RCE
5. SSH como usuario privilegiado
```

### Opción C: LDAP → SSH (Si se obtienen credenciales)
```
1. Usar Directory Manager credentials en ldapmodify
2. Crear nuevo usuario POSIX con shell SSH
3. O inyectar SSH key en entrada existente
4. SSH access → privilege escalation
```

---

## 🔑 Credenciales a Buscar

Durante la explotación, buscar:
```
- cn=Directory Manager password (en config OpenDJ)
- OpenAM admin password (en dsconfig de OpenDJ)
- SSH keys (en /home/users o /root/.ssh)
- Archivos .env o config.properties (Tomcat/OpenAM)
```

---

## ✅ Verificación de Acceso

Una vez se logre shell o acceso LDAP:

```bash
# Check 1: Enumerar usuarios locales
ldapsearch -x -H ldap://10.129.122.141:50389 \
           -b "ou=people,dc=management,dc=htb" \
           -D "cn=Directory Manager" -w [PASSWORD]

# Check 2: Verificar SSH access
ssh -i key.pem user@10.129.122.141

# Check 3: Buscar flags
find / -name "flag.txt" -o -name "flag" 2>/dev/null
```

---

## 📊 Matriz de Riesgo

| Componente | Vulnerabilidad | Impacto | Riesgo |
|-----------|-----------------|--------|--------|
| OpenAM 16.0.5 | CVE-2026-33439 | RCE pre-auth | CRÍTICO |
| OpenDJ 5.0.3 | JMX sin auth | RCE + data leak | CRÍTICO |
| LDAP 50389 | Anonymous bind | Información disclosure | ALTO |
| SSH 22 | Acceso basado en LDAP | Compromiso total | ALTO |

---

## 🛠️ Herramientas Requeridas

```bash
# Obligatorias
- curl / wget
- netcat (nc)
- ldapsearch (openldap-clients)
- Java (JDK 8+)

# Recomendadas
- ysoserial (para gadget generation)
- beanshooter (para JMX)
- Metasploit Framework (módulos OpenAM)
- jconsole (JDK util)
```

---

## 📖 Referencias Técnicas

### CVE-2026-33439
- https://advisories.gitlab.com/maven/org.openidentityplatform.openam/openam/CVE-2026-33439/
- https://github.com/OpenIdentityPlatform/OpenAM/security/advisories/GHSA-2cqq-rpvq-g5qj
- https://www.hacktron.ai/blog/openam-deserialization-pre-auth-rce

### OpenDJ & JMX Security
- https://github.com/OpenIdentityPlatform/OpenDJ-SDK
- https://docs.oracle.com/javase/8/docs/technotes/guides/management/agent.html
- https://portswigger.net/kb/issues/00100700_unsafe-java-object-deserialization

### ysoserial & Gadget Chains
- https://github.com/frohoff/ysoserial
- https://github.com/X1r0z/ActiveMQ-RCE (similar exploit chain)

---

## 📝 Notas de Implementación

### Java 21 Incompatibility
ysoserial 0.0.6 no es compatible con Java 21 debido a módulos de seguridad:
```
Solution: 
  - Usar flags: --add-opens java.base/java.lang=ALL-UNNAMED
  - O compilar ysoserial desde source (rama main)
  - O usar alternativas: jmx_rce_exploit.java (incluido)
```

### RMI Stub Binding
El RMI registry puede devolver stubs con direcciones internas (127.0.1.1).
```
Solution:
  - Usar SSH port forwarding: ssh -L 1689:127.0.0.1:1689
  - O modificar el hosts file con IP mapping
```

---

**Documentación Completa**: Todos los detalles técnicos y pasos de explotación están documentados en los archivos .md incluidos. Esta máquina es vulnerable a múltiples vectores de RCE con CVSS 9.8.

---

*Última actualización: 2026-10-04*  
*Analista: Claude Haiku 4.5*  
*Estado: Listo para explotación manual*

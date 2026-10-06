# 📚 Índice de Documentación - Management (HackTheBox)

## 🎯 Por Dónde Empezar

### 1. **Primero Lee**: `README-EXPLOTACION.md`
Resumen ejecutivo de las vulnerabilidades encontradas
- ✓ Qué corre en la máquina (OpenAM 16.0.5 + OpenDJ 5.0.3)
- ✓ Qué puertos están abiertos y por qué
- ✓ Dos vulnerabilidades críticas identificadas
- ✓ Flujos de ataque recomendados
- ⏱️ Tiempo de lectura: 10-15 minutos

---

## 🔍 Documentos de Explotación Detallados

### 2. **CVE-2026-33439-EXPLOIT-GUIDE.md**
Guía técnica para explotar OpenAM vía `jato.clientSession`

**Contenido:**
- Análisis técnico del root cause (ApplicationObjectInputStream sin whitelist)
- Diferencias con CVE-2021-35464 (por qué pasó el bypass)
- Preparación del entorno (ysoserial, Java, etc.)
- **Pasos MANUALES paso a paso** con comandos exactos
- Troubleshooting detallado
- Quick-start (4 comandos para obtener shell rápido)

**Cuándo usar:**
- Quieres explotar OpenAM directamente
- Tienes Java instalado
- Necesitas máxima claridad en cada paso

**Vulnerabilidad explorada:** CVE-2026-33439 (CVSS 9.8)

---

### 3. **EXPLOIT-JMX-ALTERNATIVE.md**
Explotación alternativa vía JMX (puertos 1689 → 39773)

**Contenido:**
- Por qué JMX es alternativa viable
- Análisis técnico: cómo OpenDJ expone JMX sin auth
- Diferencia entre acceso web y acceso JMX
- Código Java personalizado (cliente JMX que enumera MBeans)
- Cómo buscar vulnerabilidades en MBeans
- Post-explotación: acceso a LDAP y SSH

**Cuándo usar:**
- CVE-2026-33439 no funciona (errores con ysoserial)
- Quieres atacar una superficie más directa
- Necesitas acceso a la configuración interna de OpenDJ

**Vulnerabilidad explorada:** JMX expuesto sin autenticación

---

### 4. **QUICK-EXPLOIT-STEPS.sh**
Script ejecutable con todos los pasos de explotación

**Contenido:**
- Verificación de target
- Confirmación de vulnerabilidad
- Descarga de ysoserial
- Generación automática de payload
- Envío del exploit
- Instructions para obtener shell

**Cuándo usar:**
- Necesitas agilidad
- Quieres copiar y pegar comandos
- Prefieres automatización parcial

**Advertencia:** Lee cada sección antes de ejecutar

---

### 5. **jmx_rce_exploit.java**
Código fuente del cliente JMX

**Contenido:**
- Cliente Java que se conecta al RMI registry
- Enumera todos los MBeans disponibles
- Sin dependencias externas (solo JDK)
- Demuestra que la conexión no requiere autenticación

**Cuándo usar:**
```bash
javac jmx_rce_exploit.java
java jmx_rce_exploit 10.129.122.141 1689
```

---

## 🗺️ Flujo de Uso Recomendado

```
┌─────────────────────────────────────┐
│ 1. README-EXPLOTACION.md            │ ← Empieza aquí
│    (Entiende qué hay)               │
└──────────────┬──────────────────────┘
               │
     ┌─────────┴─────────┐
     │                   │
     ▼                   ▼
┌─────────────────┐  ┌──────────────────┐
│ CVE-2026-33439  │  │ JMX Alternative  │
│ (OpenAM Web)    │  │ (Puertos 1689)   │
└────────┬────────┘  └────────┬─────────┘
         │                    │
         ▼                    ▼
    Obtener shell TOMCAT / Enumerar MBeans
         │                    │
         └────────┬───────────┘
                  │
                  ▼
         Acceso a LDAP (50389/4444)
         Credenciales Directory Manager
                  │
                  ▼
         Modificar usuarios / SSH keys
                  │
                  ▼
         SSH access como usuario privilegiado
```

---

## 🧪 Casos de Uso Específicos

### Caso 1: "Necesito explotar esto AHORA"
→ Usa `QUICK-EXPLOIT-STEPS.sh`
```bash
chmod +x QUICK-EXPLOIT-STEPS.sh
./QUICK-EXPLOIT-STEPS.sh
```

### Caso 2: "Necesito entender qué está pasando"
→ Lee `README-EXPLOTACION.md` + `CVE-2026-33439-EXPLOIT-GUIDE.md`

### Caso 3: "ysoserial no funciona, necesito alternativa"
→ Usa `EXPLOIT-JMX-ALTERNATIVE.md` + `jmx_rce_exploit.java`

### Caso 4: "Quiero aprender Java + RMI"
→ Estudia `jmx_rce_exploit.java` + ejecuta manualmente

### Caso 5: "Necesito documentación para presentar"
→ Toda la documentación está lista en formato Markdown

---

## 📋 Checklist de Explotación

```
Previo:
[ ] Confirmar /etc/hosts tiene sso.management.htb
[ ] Confirmar conectividad a puerto 443
[ ] Descargar/instalar ysoserial o usar Java RMI

Explotación CVE-2026-33439:
[ ] Generar payload con ysoserial + gadget chain
[ ] Codificar en base64
[ ] Preparar listener (nc -lvnp 4444)
[ ] Enviar GET/POST con jato.clientSession
[ ] Recibir reverse shell como tomcat

Explotación JMX (Alternativa):
[ ] Compilar jmx_rce_exploit.java
[ ] Conectar a puerto 1689
[ ] Enumerar MBeans
[ ] Buscar operaciones explotables

Post-Explotación:
[ ] Buscar Directory Manager password
[ ] Conectar a LDAP (puerto 50389 o 4444)
[ ] Obtener lista de usuarios
[ ] Crear/modificar usuario con SSH access
[ ] SSH a máquina como usuario nuevo

Flag:
[ ] Ubicación típica: /root/flag o /home/user/flag
[ ] O en variable de entorno HTB_FLAG
```

---

## 🔧 Herramientas Necesarias

```bash
# Básicas (probablemente ya instaladas)
curl
nc (netcat)
ssh
bash

# Necesarias para explotación
java (JDK 8+)
ysoserial (descargar o compilar)

# Opcional pero recomendado
ldapsearch (openldap-clients)
jconsole (viene con JDK)
```

---

## 🚨 Problemas Comunes y Soluciones

| Problema | Causa | Solución |
|----------|-------|----------|
| "ysoserial: module does not open" | Java 21+ | Usa flags `--add-opens` |
| "Connection refused" en RMI | RMI bind a 127.0.1.1 | Usa SSH port forwarding |
| "Payload is empty" | ysoserial no genera | Usa CommonsCollections6 en vez de 5 |
| "No connection on nc listener" | IP attacker incorrecta | Verificar IP real: `ip addr show tun0` |
| "403 Forbidden en endpoint JATO" | Sin session | Obtén session primero con curl -c cookies.txt |

---

## 📊 Resumen Técnico

| Aspecto | Detalles |
|--------|----------|
| **Vulnerabilidad Principal** | CVE-2026-33439 (OpenAM 16.0.5) |
| **Severidad** | CVSS 9.8 (Crítica) |
| **Tipo** | Pre-Auth RCE via deserialization |
| **Alternativa** | JMX RCE sin auth (puertos 1689→39773) |
| **Componentes** | nginx + OpenAM 16.0.5 + OpenDJ 5.0.3 |
| **Usuarios objetivo** | tomcat (web application) |
| **Escalación** | OpenDJ + LDAP modification → SSH |

---

## 💾 Archivos Disponibles

```
/home/javlabs/hackthebox/Management/

├── README-EXPLOTACION.md              (Resumen ejecutivo) ⭐ EMPIEZA AQUÍ
├── CVE-2026-33439-EXPLOIT-GUIDE.md    (Guía OpenAM)
├── EXPLOIT-JMX-ALTERNATIVE.md         (Guía JMX)
├── QUICK-EXPLOIT-STEPS.sh             (Script automático)
├── jmx_rce_exploit.java               (Código Java)
├── INDICE-DOCUMENTACION.md            (Este archivo)
│
├── ports.txt                          (Nmap scan original)
├── page.html                          (SPA frontend)
├── app_decrypted.js                   (Código JavaScript desencriptado)
└── (otros archivos de análisis)
```

---

## 📞 Preguntas Frecuentes

**P: ¿Por dónde empiezo?**  
R: Lee `README-EXPLOTACION.md` primero (10 min). Luego elige OpenAM o JMX.

**P: ¿Cuál es el exploit más probable que funcione?**  
R: CVE-2026-33439 si tienes Java. JMX si Java falla.

**P: ¿Necesito estar en HTB VPN?**  
R: Sí. Necesitas IP 10.10.x.x y conectividad a 10.129.122.141

**P: ¿Qué pasa si tengo errores con ysoserial?**  
R: Lee la sección "Java 21 Incompatibility" en README. O usa JMX.

**P: ¿Cómo busco el flag?**  
R: Una vez con shell: `find / -name "*flag*" 2>/dev/null` o `/root/flag`

---

## ✅ Estado

- ✓ Vulnerabilidades identificadas
- ✓ Análisis técnico completo
- ✓ Documentación paso a paso
- ✓ Código fuente de exploits
- ✓ Troubleshooting incluido
- ✓ Listo para explotación manual

---

**Última actualización:** 2026-10-04  
**Documentación:** Completamente documentada para reproducción manual  
**Nivel de detalle:** De muy bajo (executive summary) a muy alto (línea por línea)

Selecciona uno de los documentos arriba y comienza. ¡Buena suerte! 🚀

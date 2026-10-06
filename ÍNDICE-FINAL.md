# 📚 Índice Completo - Management HTB (CVE-2026-33439)

## 🎯 Flujo Completo de Explotación

```
┌─────────────────────────────────────────────────────────┐
│ INICIO: Management HTB (10.129.122.141)                 │
│ Vulnerabilidad: CVE-2026-33439 (OpenAM 16.0.5)          │
└────────────────┬────────────────────────────────────────┘
                 │
    ┌────────────▼────────────────┐
    │  1. INFORMACIÓN PREVIA       │
    │  README-EXPLOTACION.md      │
    │  INDICE-DOCUMENTACION.md    │
    └────────────┬────────────────┘
                 │
    ┌────────────▼────────────────┐
    │  2. OBTENER RCE             │
    │  EXPLOIT-PASO-A-PASO.md     │  ← MEJOR GUÍA
    │  RUN-EXPLOIT.sh             │  ← EJECUTAR AQUÍ
    │  CVE-2026-33439-REAL-EXPLOIT/  (+ JARs)
    └────────────┬────────────────┘
                 │
                 ▼
         Shell como 'openam'
         (uid=996, gid=987)
                 │
    ┌────────────▼────────────────┐
    │  3. ESCALAR PRIVILEGIOS     │
    │  POST-EXPLOITATION-...md    │  ← ESTÁS AQUÍ
    │  - Buscar credenciales LDAP │
    │  - Modificar usuarios       │
    │  - Acceso SSH como owen     │
    └────────────┬────────────────┘
                 │
                 ▼
         Flag obtenida (root o owen)
         ✓ MÁQUINA COMPROMETIDA
```

---

## 📂 Documentos por Fase

### FASE 0: Reconnaissance
- **README-EXPLOTACION.md** (10 min read)
  - Vulnerabilidades identificadas
  - Componentes de la máquina
  - Rutas de ataque disponibles

### FASE 1: Explotación
- **EXPLOIT-PASO-A-PASO.md** ⭐ RECOMENDADO
  - Paso a paso basado en código REAL
  - Comandos exactos
  - Troubleshooting incluido

- **RUN-EXPLOIT.sh** (Script automatizado)
  - Verifica prerequisites
  - Ejecuta exploit automáticamente
  - Proporciona feedback
  ```bash
  bash RUN-EXPLOIT.sh
  ```

- **CVE-2026-33439-EXPLOIT-GUIDE.md** (Alternativa teórica)
  - Análisis técnico profundo
  - Gadget chain explicado
  - Quick-start de 4 comandos

- **CVE-2026-33439-REAL-EXPLOIT/** (Repositorio)
  - Script Python: `02 Exploit/Exploit_CVE_2026_33439.py`
  - JARs necesarios: `02 Exploit/Jars/` (7 archivos)
  - README oficial

### FASE 2: Post-Explotación (AHORA AQUÍ)
- **POST-EXPLOITATION-PRIVILEGE-ESCALATION.md** 🔴 ACTIVO
  - Fase 1: Enumeración inicial
  - Fase 2: Buscar credenciales LDAP
  - Fase 3: Modificar LDAP para escalar
  - Fase 4: Acceso SSH como owen
  - Fase 5: Obtener flag
  - Rutas alternativas si algo falla

### ALTERNATIVAS
- **EXPLOIT-JMX-ALTERNATIVE.md**
  - Si CVE-2026-33439 falla
  - Atacar JMX en puerto 1689

---

## 🚀 Comando Rápido para Empezar AHORA

```bash
# Terminal 1 (Listener):
nc -lvnp 4444

# Terminal 2 (Ataque):
cd /home/javlabs/hackthebox/Management
bash RUN-EXPLOIT.sh
```

Selecciona **Y** cuando pregunte si iniciaste el listener.

---

## ⚡ Próximos Pasos (Post-RCE)

Ahora que tienes shell como `openam`:

```bash
# Abre POST-EXPLOITATION-PRIVILEGE-ESCALATION.md

# Ejecuta estos comandos EN TU SHELL:
find / -name "*flag*" -type f 2>/dev/null | grep -v proc
grep -r "password" /opt/openam-tomcat/webapps/openam/WEB-INF/ 2>/dev/null | head -20
ldapsearch -x -H ldap://localhost:50389 -b "ou=people,dc=management,dc=htb" "uid=*" uid
```

---

## 📊 Resumen de Archivos

| Archivo | Uso | Prioridad |
|---------|-----|-----------|
| EXPLOIT-PASO-A-PASO.md | Guía técnica completa | ⭐⭐⭐ |
| RUN-EXPLOIT.sh | Ejecutar exploit | ⭐⭐⭐ |
| POST-EXPLOITATION-PRIVILEGE-ESCALATION.md | Escalar privilegios | ⭐⭐⭐ |
| README-EXPLOTACION.md | Entender contexto | ⭐⭐ |
| CVE-2026-33439-EXPLOIT-GUIDE.md | Aprender técnicamente | ⭐⭐ |
| EXPLOIT-JMX-ALTERNATIVE.md | Plan B si falla | ⭐ |
| CVE-2026-33439-REAL-EXPLOIT/ | Código fuente | ⭐ |

---

## 🔍 Troubleshooting Rápido

| Problema | Solución |
|----------|----------|
| "javac: command not found" | `sudo apt-get install openjdk-21-jdk` |
| "No reachable JATO endpoints" | Verifica `/etc/hosts` tiene `sso.management.htb` |
| "Reverse shell conecta pero está muerta" | Usa comandos simples, no interactivos |
| "No encuentro credenciales" | Sigue POST-EXPLOITATION-PRIVILEGE-ESCALATION.md sección Alternativas |
| "LDAP modification falla" | Verifica que tienes la password correcta del Directory Manager |

---

## ✅ Checklist de Progreso

- [ ] Entender la vulnerabilidad (READ: README-EXPLOTACION.md)
- [ ] Ejecutar exploit (RUN: RUN-EXPLOIT.sh)
- [ ] Obtener shell como `openam`
- [ ] Ejecutar enumeración inicial (READ: POST-EXPLOITATION-PRIVILEGE-ESCALATION.md Fase 1)
- [ ] Encontrar credenciales LDAP (Fase 2)
- [ ] Escalar privilegios (Fase 3)
- [ ] Obtener acceso SSH como `owen` (Fase 4)
- [ ] Leer la flag (Fase 5)
- [ ] ✓ MÁQUINA COMPROMETIDA

---

## 🎓 Lo que Aprendiste

✓ Deserialization RCE en Java (CVE-2026-33439)  
✓ Gadget chains (PriorityQueue → TemplatesImpl)  
✓ Exfiltración de datos sin feedback directo  
✓ Escalación vía LDAP modification  
✓ OpenAM/OpenDJ internals  

---

**Estado:** Listo para continuar post-explotación  
**Siguiente paso:** Lee POST-EXPLOITATION-PRIVILEGE-ESCALATION.md y ejecuta Fase 1


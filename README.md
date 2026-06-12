# DTP VLAN Hopping Attack

**Autor:** Edwin (Matrícula: 2024-2415)  
**Institución:** Instituto Tecnológico de las Américas (ITLA)  
**Curso:** Seguridad de Redes  
**Fecha:** 12 de Junio de 2026

---

## Descripción General

Este repositorio contiene scripts, documentación y demostración de vulnerabilidades en el protocolo DTP (Dynamic Trunking Protocol) de Cisco. Se demuestra cómo un atacante puede convertir un puerto de acceso a trunk sin autorización, obteniendo acceso a todas las VLANs de la red.

**Ataque demostrado:**
- Convertir un puerto de acceso (access) a puerto troncal (trunk) mediante DTP negotiation spoofing

---

## Objetivo del Laboratorio

Demostrar las vulnerabilidades inherentes del protocolo DTP en equipos Cisco:

1. DTP confía ciegamente en los frames de negotiation sin validar origen
2. No existe validación de autenticidad del origen del paquete
3. Un atacante puede cambiar el modo de un puerto sin autorización
4. El acceso a un puerto es suficiente para comprometer la segmentación de VLANs

---

## Videos Demostrativos

**Video:** Demostración práctica de DTP VLAN Hopping Attack

[Ver en YouTube](https://youtu.be/tFmJXBdxJt8)   
Contenido: Topología, explicación, ejecución del ataque, verificación del cambio de modo

---

## Requisitos

### Software
- Linux (Parrot OS, Kali, Ubuntu, etc.)
- Yersinia (herramienta de ataques L2)
- SSH o consola del switch

### Hardware
- Interfaz de red conectada a un puerto del switch
- Acceso a switches Cisco con DTP habilitado (default)

### Instalación de dependencias

**Instalar Yersinia:**

```bash
sudo apt update
sudo apt install yersinia -y
```

O compilar desde fuente:

```bash
git clone https://github.com/tomac/yersinia.git
cd yersinia
./autogen.sh
./configure --disable-gtk
make
sudo make install
```

---

## Uso Rápido

### Con Yersinia (Recomendado)

```bash
TERM=xterm sudo yersinia -I
```

Dentro de Yersinia:
1. Selecciona protocolo DTP
2. Elige "Launch attack" o "Send DTP frames"
3. Ejecuta el ataque
4. Verifica el cambio de puerto en el switch

### Verificación en el Switch

```cisco
SW-CORE# show interfaces Et0/2 switchport

Administrative Mode: dynamic auto
Operational Mode: static access          (ANTES)
```

Después del ataque:

```cisco
SW-CORE# show interfaces Et0/2 switchport

Administrative Mode: dynamic auto
Operational Mode: trunk                  (DESPUÉS)
Trunking VLANs Enabled: ALL
```

---

## Fundamentos Técnicos

### Protocolo DTP

DTP es un protocolo propietario de Cisco que permite a los switches negociar automáticamente el modo de un puerto (access o trunk).

**Modos de Negotiation:**

| Modo | Comportamiento |
|------|----------------|
| Desirable | Intenta activamente volverse trunk |
| Auto | Acepta volverse trunk si lo solicitan |
| On | Fuerza trunk (sin negotiation) |
| Off/Access | Fuerza access (deshabilita DTP) |
| NonNegotiate | Modo static sin negotiation |

### Mecanismo de Vulnerabilidad

Los switches aceptan DTP frames sin validar la autenticidad del origen:

```
Atacante envía DTP Desirable
    ↓
Switch recibe frame (confía ciegamente)
    ↓
Switch inicia negotiation
    ↓
Puerto cambia a TRUNK
    ↓
Acceso a TODAS las VLANs
```

### Flujo del Ataque

1. Identificar puerto en modo access con DTP habilitado
2. Usar Yersinia para generar DTP frames
3. Enviar modo "Desirable" (solicita trunk)
4. Switch responde y cambia a trunk
5. Atacante obtiene acceso a todas las VLANs

---

## Contra-Medidas

### Nivel 1: DTP Disable (Recomendado)

```cisco
SW(config-if)# switchport nonegotiate
```

Desactiva negotiation completamente. El puerto se queda en modo configurado.

**Efectividad:** Muy Alta  
**Complejidad:** Muy Baja

### Nivel 2: Access Mode Forzado

```cisco
SW(config-if)# switchport mode access
```

Força modo access sin permitir negotiation.

**Efectividad:** Alta  
**Complejidad:** Baja

### Nivel 3: BPDU Guard

```cisco
SW(config-if)# spanning-tree bpduguard enable
```

Detecta actividad STP anómala en puertos access.

**Efectividad:** Media (previene algunos ataques relacionados)  
**Complejidad:** Media

### Nivel 4: Port Security

```cisco
SW(config-if)# switchport port-security
```

Limita quién puede conectarse al puerto.

**Efectividad:** Alta (previene acceso no autorizado)  
**Complejidad:** Media

### Mejor Práctica

**Configuración recomendada para puertos access:**

```cisco
interface Ethernet0/2
  switchport mode access
  switchport access vlan 1
  switchport nonegotiate
  spanning-tree bpduguard enable
  switchport port-security
  switchport port-security maximum 1
end
```

---

## Topología del Laboratorio

```
┌─────────────────┐
│   PARROT OS     │
│ 172.24.15.129   │ Atacante
└────────┬────────┘
         │ (access port Et0/2)
         │
┌────────▼────────┐         ┌──────────────┐
│  SW-CORE        │ Et0/1──┤  SW-ACCESO   │
│  172.24.15.24   │────────┤  172.24.15.15│
└─────────────────┘         └──────────────┘

DTP Domain: ITLA-NET
VLANs: 1 (default), 15 (VLAN_USERS), 24 (VLAN_MGMT)
```

---

## Impacto

### Antes del Ataque
- Puerto Et0/2 en access mode
- Limitado a VLAN 1 (default)
- Acceso restringido

### Después del Ataque
- Puerto Et0/2 en trunk mode
- Acceso a TODAS las VLANs
- Visibilidad total del tráfico
- VLAN hopping posible

### Consecuencias
- Violación de segmentación de red
- Acceso a datos confidenciales en otras VLANs
- Sniffing de tráfico inter-VLAN
- Man-in-the-middle entre VLANs

---

## Limitaciones Conocidas

- DTP Disable: Si está configurado, el ataque no funciona
- VTP Mode Transparent: Aísla cambios localmente
- Port Security: Puede prevenir acceso al puerto
- El atacante necesita acceso físico (o inalámbrico) a un puerto

---

## Referencias

- Cisco DTP (Dynamic Trunking Protocol) Documentation
- IEEE 802.1Q (VLAN Tagging Specification)
- Yersinia Documentation: https://github.com/tomac/yersinia
- Network Penetration Testing Frameworks
- "VLAN Attacks" - Advanced Network Security

---

## Notas Importantes

Este código es para fines educativos exclusivamente. Solo para uso en laboratorios autorizados de ITLA.

Prohibido usar en redes de producción o sin autorización explícita.

---

## Autor

Edwin  
Matrícula: 2024-2415  
Programa: Seguridad Informática  
Instituto Tecnológico de las Américas (ITLA)  

---

**Última actualización:** 12 de Junio de 2024  
**Versión:** 1.0  
**GitHub:** https://github.com/edwindepaula10/8_dtp_vlan_hooping

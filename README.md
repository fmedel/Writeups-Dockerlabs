# Máquinas - Writeups

## Tabla de Contenidos

- [Estructura del Proyecto](#estructura-del-proyecto)
- [Máquinas Disponibles](#máquinas-disponibles)
- [Muy Fácil](#muy-fácil)
  - [BorazuwarahCTF](#borazuwarahctf)
  - [BreakMySSH](#breakmyssh)
  - [Firsthacking](#firsthacking)
  - [Hedgehog](#hedgehog)
  - [Obsession](#obsession)
  - [Tproot](#tproot)
  - [Trust](#trust)
  - [Vacaciones](#vacaciones)
- [Fácil](#fácil)
  - [Injection](#injection)
  - [Psycho](#psycho)

---

## Estructura del Proyecto

```
maquinas
├── README.md                          # Este archivo - Índice de todas las máquinas
├── Muy_Fácil/
│   ├── BorazuwarahCTF/
│   │   └── README.md                  # Writeup completo de BorazuwarahCTF
│   ├── BreakMySSH/
│   │   └── README.md                  # Writeup completo de BreakMySSH
│   ├── Firsthacking/
│   │   └── README.md                  # Writeup completo de Firsthacking
│   ├── Hedgehog/
│   │   └── README.md                  # Writeup completo de Hedgehog
│   ├── Obsession/
│   │   └── README.md                  # Writeup completo de Obsession
│   ├── Tproot/
│   │   ├── README                     # Writeup completo de Tproot
│   │   └── exploit_vsftpd.py          # Exploit personalizado para vsftpd 2.3.4
│   ├── Trust/
│   │   └── README.md                  # Writeup completo de Trust
│   └── Vacaciones/
│       └── README.md                  # Writeup completo de Vacaciones
└── Fácil/
    ├── Injection/
    │   ├── README.md                  # Writeup completo de Injection
    │   └── exploit_sqli.py            # Exploit SQL Injection personalizado
    └── Psycho/
        └── README.md                  # Writeup completo de Psycho
```

---

## Máquinas Disponibles

| Dificultad | Máquina | Fecha del Writeup | CVE |
|------------|---------|-------------------|-----|
| Muy Fácil | [BorazuwarahCTF](Muy_Fácil/BorazuwarahCTF/) | 10/12/2025 | - |
| Muy Fácil | [BreakMySSH](Muy_Fácil/BreakMySSH/) | 10/12/2025 | CVE-2018-15473 |
| Muy Fácil | [Firsthacking](Muy_Fácil/Firsthacking/) | 08/12/2025 | CVE-2011-2523 |
| Muy Fácil | [Hedgehog](Muy_Fácil/Hedgehog/) | 11/12/2025 | - |
| Muy Fácil | [Obsession](Muy_Fácil/Obsession/) | 15/12/2025 | - |
| Muy Fácil | [Tproot](Muy_Fácil/Tproot/) | 10/12/2025 | CVE-2011-2523 |
| Muy Fácil | [Trust](Muy_Fácil/Trust/) | 07/12/2025 | - |
| Muy Fácil | [Vacaciones](Muy_Fácil/Vacaciones/) | 12/12/2025 | - |
| Fácil | [Injection](Fácil/Injection/) | 18/12/2025 | - |
| Fácil | [Psycho](Fácil/Psycho/) | 15/12/2025 | - |

---
## Muy Fácil

### [BorazuwarahCTF](Muy_Fácil/BorazuwarahCTF/)
Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas que incluyen exposición de metadatos en imágenes, contraseñas débiles y permisos sudo mal configurados.

- **Dificultad:** Muy Fácil
- **Autor:** [borazuwarah](https://dockerlabs.es/#)
- **CVE:** -

---

### [BreakMySSH](Muy_Fácil/BreakMySSH/)
Máquina CTF enfocada en la explotación de vulnerabilidades en servicios SSH. Esta réplica implementa OpenSSH 7.7, vulnerable al CVE-2018-15473 que permite enumeración de usuarios válidos.

- **Dificultad:** Muy Fácil
- **Autor:** [El Pingüino de Mario](https://github.com/Maalfer)
- **CVE:** CVE-2018-15473

---

### [Firsthacking](Muy_Fácil/Firsthacking/)
Máquina CTF diseñada como introducción al pentesting, enfocada en la explotación de servicios vulnerables conocidos. Esta réplica implementa la versión comprometida de vsftpd 2.3.4, que contiene una backdoor histórica descubierta en 2011.

- **Dificultad:** Muy Fácil
- **Autor:** [El Pingüino de Mario](https://dockerlabs.es/#)
- **CVE:** CVE-2011-2523

---

### [Hedgehog](Muy_Fácil/Hedgehog/)
Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas de configuraciones inseguras que incluyen exposición de información sensible, contraseñas débiles y escalada de privilegios mediante configuraciones sudo mal establecidas con usuarios intermedios.

- **Dificultad:** Muy Fácil
- **Autor:** [AnkbNikas](https://github.com/AnkbNikas)
- **CVE:** -

---

### [Obsession](Muy_Fácil/Obsession/)
Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas de configuraciones inseguras que incluyen exposición de información sensible a través de comentarios HTML, contraseñas débiles y permisos sudo mal configurados.

- **Dificultad:** Muy Fácil
- **Autor:** [Juan](https://dockerlabs.es/#)
- **CVE:** -

---

### [Tproot](Muy_Fácil/Tproot/)
Máquina CTF diseñada como introducción al pentesting, enfocada en la explotación de servicios vulnerables conocidos. Esta réplica implementa la versión comprometida de vsftpd 2.3.4, que contiene una backdoor histórica descubierta en 2011.

- **Dificultad:** Muy Fácil
- **Autor:** [d1se0](https://github.com/D1se0)
- **CVE:** CVE-2011-2523

---

### [Trust](Muy_Fácil/Trust/)
Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas de configuraciones inseguras que incluyen exposición de información sensible, contraseñas débiles y permisos sudo mal configurados.

- **Dificultad:** Muy Fácil
- **Autor:** [El Pingüino de Mario](https://dockerlabs.es/#)
- **CVE:** -

---

### [Vacaciones](Muy_Fácil/Vacaciones/)
Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas de configuraciones inseguras que incluyen exposición de información sensible en código HTML, contraseñas débiles, movimiento lateral entre usuarios y permisos sudo mal configurados.

- **Dificultad:** Muy Fácil
- **Autor:** [Romabri](https://github.com/romabri)
- **CVE:** -

---
## Fácil

### [Injection](Fácil/Injection/)
Máquina CTF diseñada para practicar técnicas de inyección SQL y escalada de privilegios mediante binarios SUID. Esta máquina implementa un formulario de login vulnerable a SQL injection que permite extraer credenciales de la base de datos, seguido de una escalada de privilegios mediante el binario `/usr/bin/env` con permisos SUID.

- **Dificultad:** Fácil
- **Autor:** [El Pingüino de Mario](https://dockerlabs.es/#)
- **CVE:** -

---

### [Psycho](Fácil/Psycho/)
Máquina CTF diseñada para practicar técnicas de pentesting básicas en un entorno controlado. Esta réplica implementa vulnerabilidades típicas que incluyen Local File Inclusion (LFI), exposición de claves SSH privadas y técnicas de escalada de privilegios mediante Python library hijacking.

- **Dificultad:** Fácil
- **Autor:** [Luisillo_o](https://dockerlabs.es/#)
- **CVE:** -

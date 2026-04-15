# WinRAR Keygen — Generador de Claves

> Proyecto de investigación y análisis de seguridad con fines educativos.
> Permite estudiar el algoritmo de licenciamiento de WinRAR basado en criptografía de curvas elípticas (ECC).

[![GitHub Pages](https://img.shields.io/badge/demo-live-brightgreen?style=flat-square&logo=github)](https://joshegatito.github.io/winrar/)
[![License](https://img.shields.io/badge/license-Educational-blue?style=flat-square)]()
[![Languages](https://img.shields.io/badge/languages-HTML%20%7C%20JS%20%7C%20WebAssembly-orange?style=flat-square)]()

---

## Demo

[https://joshegatito.github.io/winrar/](https://joshegatito.github.io/winrar/)

---

## ¿Qué hace este proyecto?

Genera el archivo `rarreg.key` necesario para registrar WinRAR. El archivo contiene:

- **Nombre de usuario** — identificador de la licencia
- **Tipo de licencia** — etiqueta visible en "Acerca de WinRAR"
- **UID** — identificador único derivado criptográficamente del nombre de usuario
- **Firma ECC** — firma digital que WinRAR valida para autenticar la licencia

Una vez generado, basta con copiar el archivo `rarreg.key` a la carpeta de instalación de WinRAR o a `%APPDATA%\WinRAR`.

---

## ¿Cómo funciona internamente?

WinRAR utiliza **criptografía de curvas elípticas (ECC)** sobre un campo binario GF(2^n) para validar sus licencias. El proceso de generación sigue estos pasos:

```
Nombre de usuario
      │
      ▼
   Hash → Escalar k
      │
      ▼
   k × G = Punto público (UID)        ← determinista, mismo usuario = mismo UID
      │
      ▼
   ECDSA (nonce aleatorio)
      │
      ▼
   Firma (r, s)                       ← varía en cada generación
      │
      ▼
   Formato rarreg.key
```

El motor criptográfico está compilado en **WebAssembly** (`keygen.wasm`) usando **Emscripten** desde código C++ que incluye una build completa de **OpenSSL**. La clave privada EC está embebida en el binario compilado.

---

## Estructura del archivo `rarreg.key`

```
RAR registration data
<nombre de usuario>
<tipo de licencia>
UID=<20 chars hex — 80 bits — punto público EC>
<7 líneas × 54 chars hex — firma ECDSA (189 bytes total)>
```

### Desglose de la firma (189 bytes)

| Bytes | Contenido | ¿Varía? |
|-------|-----------|---------|
| 0–4   | Header de versión (`6412212250`) | No |
| 5–36  | Punto público del usuario (ECC) | Por usuario |
| 37    | Separador `0x60` | No |
| 38–98 | Componente `r` de la firma ECDSA | Sí (nonce) |
| 99    | Separador `0x60` | No |
| 100–159 | Componente `s` de la firma ECDSA | Sí (nonce) |
| 160   | Separador `0x60` | No |
| 161–188 | Hash/tail del usuario | Por usuario |

---

## Tecnologías

| Archivo | Tecnología | Descripción |
|---------|-----------|-------------|
| `index.html` | HTML / CSS / JS | Interfaz de usuario |
| `keygen.js` | JavaScript (Emscripten) | Glue code — puente entre el browser y el WASM |
| `keygen.wasm` | WebAssembly (C++ + OpenSSL) | Motor criptográfico ECC (~967 KB) |

---

## Uso

### Opción A — GitHub Pages (sin instalar nada)

```
https://joshegatito.github.io/winrar/
```

### Opción B — Servidor local (Laragon / XAMPP)

```
http://localhost/winrar/
```

> **Nota:** El archivo `keygen.wasm` requiere ser servido por HTTP. No funciona abriendo `index.html` directamente desde el explorador de archivos (`file://`).

---

## Aviso legal

Este proyecto es de uso **estrictamente educativo** para el estudio de:

- Criptografía de curvas elípticas (ECC)
- Formato de licencias de software
- Compilación C++ a WebAssembly con Emscripten
- Ingeniería inversa de binarios

El uso de este software para eludir licencias comerciales puede violar los términos de uso de WinRAR y las leyes de propiedad intelectual de tu país. El autor no se responsabiliza del uso indebido.

---

## Referencias

- [WebAssembly — MDN](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [Emscripten Documentation](https://emscripten.org/docs/)
- [Elliptic Curve Cryptography — Wikipedia](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography)
- [OpenSSL ECC](https://wiki.openssl.org/index.php/Elliptic_Curve_Cryptography)

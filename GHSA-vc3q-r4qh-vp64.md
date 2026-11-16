# Ed25519 en dryoc: `SigningKeyPair::from_slices` y `serde` aceptan una clave secreta Ed25519 de 64 bytes cuya clave pública embebida no coincide con la semilla

## Identificadores
- GHSA: `GHSA-vc3q-r4qh-vp64`
- CVE: aún sin asignar (al 2026-11-16)

## Proyecto afectado
- Nombre: dryoc (Rust crate)
- Repositorio o sitio: https://github.com/brndnmtthws/dryoc

## Tipo de vulnerabilidad
CWE-20 — Improper Input Validation; CWE-345 — Insufficient Verification of Data Authenticity

## Severidad
- Severidad: Low (clasificación del mantenedor en el advisory de GitHub)
- Sin vector CVSS publicado

## Versiones afectadas
<= 2.0.0 (comprobado en 0.9.0, 1.0.0 y 2.0.0)

## Versión corregida
2.0.1

## Descripción
Una clave secreta Ed25519 son `seed || public_key`, y la firma lee la clave pública de los bytes 32..64 de la clave secreta. En dryoc, `crypto_sign_ed25519_detached_impl` toma la clave pública usada en el hash del reto de `secret_key[32..]`, mientras que el nonce `r` se deriva solo de la semilla y del mensaje.

Si se reemplazan los bytes 32..64 de una clave secreta, firmar el mismo mensaje con la clave original y con la modificada produce dos firmas con el mismo `R` y distinto `S`. A partir de ahí, `a = (S1 - S2) / (k1 - k2)` recupera el escalar privado.

`SigningKeyPair::from_secret_key` re-deriva el par de claves a partir de la semilla, así que no se ve afectado. En cambio, `SigningKeyPair::from_slices` y la implementación derivada de `Deserialize` guardan los 64 bytes tal cual, sin comprobar la consistencia entre la semilla y la clave pública embebida. dryoc 2.0.0 ya endurecía un constructor contra esto, pero dejaba los otros dos abiertos.

No es un fallo de compatibilidad con libsodium: libsodium 1.0.22 se comporta igual por la misma razón (el secreto es `seed || public_key`).

## Impacto
El atacante debe poder cambiar la mitad pública de una clave secreta almacenada sin leer la semilla (por ejemplo, almacenamiento de claves sin protección de integridad, o una aplicación que arma los 64 bytes desde dos fuentes distintas) y obtener firmas del mismo mensaje con la clave original y la modificada. En ese escenario puede recuperar el escalar privado y forjar firmas válidas bajo la clave pública real de la víctima. El mantenedor y quien reporta coinciden en severidad Low.

Mitigación: actualizar a dryoc >= 2.0.1, o re-derivar la clave pública desde la semilla al cargar (`from_slices`/deserialización) y rechazar si no coincide.

## Línea de tiempo de divulgación coordinada
| Fecha | Hito |
| --- | --- |
| 2026-11-16 | Reporte privado aceptado por el mantenedor |
| 2026-11-16 | Advisory público del mantenedor (GHSA-vc3q-r4qh-vp64) |
| 2026-11-16 | Publicación en este repositorio |

## Crédito
Beatriz Domínguez Pérez (bettycrypto77-dev, EteisNet)

## Referencias
- Advisory del mantenedor: https://github.com/brndnmtthws/dryoc/security/advisories/GHSA-vc3q-r4qh-vp64
- Release parcheado: https://github.com/brndnmtthws/dryoc/releases/tag/v2.0.1

# X25519 in dryoc < 1.0.0 accepts low-order public keys (`crypto_kx_*_session_keys` returns `Ok` with session keys computable without any private key)

## Identificadores

- GHSA: `GHSA-6xqm-2982-gvrm`
- CVE: aún sin asignar (al 2026-11-16)

## Proyecto afectado

- Nombre: dryoc (Rust crate)
- Repositorio o sitio: https://github.com/brndnmtthws/dryoc

## Tipo de vulnerabilidad

CWE-20 — Improper Input Validation; CWE-325 — Missing Cryptographic Step

## Severidad (CVSS)

- Severidad: Moderate
- CVSS: `CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:L/A:N`

## Versiones afectadas

< 1.0.0

## Versión corregida

1.0.0

## Descripción

En dryoc anteriores a 1.0.0, las APIs de intercambio de claves y de multiplicación escalar X25519 aceptan claves públicas de bajo orden. En particular, `crypto_kx_client_session_keys` y `crypto_kx_server_session_keys` (y las APIs equivalentes de `kx::Session`) devuelven `Ok` y producen claves de sesión que se pueden recalcular a partir de las claves públicas, sin conocer ninguna clave privada. libsodium rechaza estas entradas. El detalle técnico consta en el advisory público del mantenedor.

## Impacto

Las aplicaciones en dryoc < 1.0.0 que usan esas APIs con una clave pública de un par no autenticada, o que un tercero puede elegir, pueden perder la confidencialidad e integridad de esa sesión, también frente a observadores pasivos. También hay una filtración menor de bits de la clave privada estática, que no reduce de forma significativa la dificultad de recuperarla. Quien solo use claves de par autenticadas generadas por dryoc o libsodium no se ve afectado en la práctica. Remediación: actualizar a dryoc >= 1.0.0. El release 1.0.0 está documentado como incompatible con 0.9.0.

## Línea de tiempo de divulgación coordinada

| Fecha | Evento |
| ----- | ------ |
| 2026-07-22 | Release parcheado (dryoc 1.0.0) |
| 2026-11-16 | Advisory público del mantenedor |
| 2026-11-16 | Publicación en este repositorio |

## Crédito

bettycrypto77-dev (EteisNet)

## Referencias

- Advisory del mantenedor: https://github.com/brndnmtthws/dryoc/security/advisories/GHSA-6xqm-2982-gvrm
- GHSA: https://github.com/brndnmtthws/dryoc/security/advisories/GHSA-6xqm-2982-gvrm
- CVE: aún sin asignar
- Release parcheado: https://github.com/brndnmtthws/dryoc/releases/tag/v1.0.0

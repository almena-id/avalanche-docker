# Red privada de Avalanche con 5 nodos

Este proyecto levanta una red privada de Avalanche usando cinco instancias de AvalancheGo dentro de Docker Compose. Se aprovecha el génesis y las llaves públicas que Avalanche Labs publica para su red "local", pero se ejecuta con un `network-id` personalizado (1337) para permitir el uso de un génesis propio con nodos ya autorizados sin necesidad de generar credenciales manualmente.

## Requisitos

- Docker 20.10+ y el plugin/CLI de Docker Compose v2.
- Al menos 8 GB de RAM libres y ~10 GB de espacio para los datos de los nodos.

## Puesta en marcha

1. Clona este repositorio y entra a la carpeta.
2. (Opcional) Ajusta la versión de `avaplatform/avalanchego` editando `.env`.
3. Levanta la red:

   ```bash
   docker compose up -d
   ```

4. Verifica que los nodos estén vivos revisando los logs o llamando al API Info, por ejemplo:

   ```bash
   curl -s http://localhost:9650/ext/info \
     -X POST \
     -H 'content-type: application/json' \
     -d '{"jsonrpc":"2.0","id":1,"method":"info.getNodeID"}'
   ```

   Repite cambiando el puerto (`9652`, `9654`, `9656`, `9658`) para los demás nodos.

5. Para detener la red:

   ```bash
   docker compose down
   ```

6. Si necesitas un reinicio limpio, elimina las carpetas bajo `data/` antes de volver a levantar los servicios.

## ¿Qué incluye cada nodo?

| Nodo | HTTP | Staking | NodeID |
|------|------|---------|--------|
| node1 | 9650 | 9651 | NodeID-7Xhw2mDxuDS44j42TCB6U5579esbSt3Lg |
| node2 | 9652 | 9653 | NodeID-MFrZFVCXPv5iCn6M9K6XduxGTYp891xXZ |
| node3 | 9654 | 9655 | NodeID-NFBbbJ4qCmNaCzeW7sxErhvWqvEQMnYcN |
| node4 | 9656 | 9657 | NodeID-GWPcbFJZFfZreETSoWjPimr846mXEKCtu |
| node5 | 9658 | 9659 | NodeID-P7oB2McjBGgW2NXXWVYjV8JEDFoW9xDE5 |

Todos los servicios comparten el génesis `artifacts/genesis/genesis.json` (idéntico al publicado para la red local) y las llaves TLS/BLS que viven en `artifacts/staking/node*/`. El `network-id` configurado es `1337`, así que no colisiona con los IDs estándar. Esas llaves **solo deben usarse para redes locales o de laboratorio**.

Cada contenedor tiene una IP fija dentro de la red bridge (`172.28.0.11`-`172.28.0.15`) para que los parámetros `bootstrap-ips` apunten a direcciones válidas.

## Estructura del proyecto

```
artifacts/
  genesis/genesis.json       # génesis local de Avalanche
  staking/node*/             # llaves TLS (staker.crt/key) y BLS (signer.key)
config/node*/config.json     # configuración individual de cada nodo
 data/node*/                 # base de datos y logs persistentes
 docker-compose.yml          # orquestación de los 5 contenedores
 .env                        # versión de AvalancheGo y nombre del proyecto Compose
 README.md
```

## Personalización rápida

- Cambia puertos o parámetros específicos editando `config/nodeX/config.json` antes de levantar los servicios.
- Puedes añadir más nodos duplicando un bloque en `docker-compose.yml`, generando/añadiendo nuevas llaves y actualizando el génesis para incluir al nuevo validador.
- Para probar upgrades basta con cambiar `AVALANCHEGO_VERSION` en `.env` y volver a aplicar `docker compose up -d`.

## Notas

- Los datos del C-Chain y del resto de chains se almacenan en `data/node*/db`; al ser una red privada, el tamaño crecerá con las pruebas que ejecutes.
- Los endpoints RPC de las chains por defecto son accesibles desde el host, por ejemplo `http://localhost:9650/ext/bc/C/rpc` para el C-Chain del nodo1.
- Si necesitas exponer los nodos fuera de tu máquina, asegúrate de ajustar `public-ip` en cada `config.json` con la IP pública deseada.

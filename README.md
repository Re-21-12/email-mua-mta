# Sistema de Correo Docker Multi-Dominio

Este proyecto configura un sistema completo de correo electrónico con múltiples dominios usando Docker Compose, incluyendo:

- 2 servidores de correo independientes (MTA1 y MTA2)
- 2 clientes webmail (Roundcube)  
- VPN WireGuard
- Enrutamiento automático entre dominios

## 🚀 Características

- ✅ **Autenticación IMAP/SMTP funcional**
- ✅ **Envío de correos sin errores** 
- ✅ **Enrutamiento entre dominios automático**
- ✅ **Volúmenes Docker optimizados** (evita problemas de permisos Windows)
- ✅ **Configuración simplificada** (servicios no esenciales deshabilitados)

## 📋 Credenciales por defecto

### MTA1 (example1.local)
- `user1@example1.local` / `password123`
- `user2@example1.local` / `password123`
- **Webmail**: http://localhost:8081

### MTA2 (example2.local)  
- `user3@example2.local` / `password123`
- `user4@example2.local` / `password123`
- **Webmail**: http://localhost:8082

## 🛠️ Instalación

1. **Clonar el repositorio:**
```bash
git clone <tu-repo-url>
cd email-docker
```

2. **Iniciar los servicios:**
```bash
docker-compose up -d
```

3. **Esperar inicialización (2-3 minutos):**
```bash
docker-compose logs -f mta1 mta2
```

4. **Acceder a los webmails:**
- MTA1: http://localhost:8081
- MTA2: http://localhost:8082

## 📁 Estructura del proyecto

```
email-docker/
├── Docker-compose.yml          # Configuración principal
├── config1/                    # Configuración MTA1
│   ├── postfix-accounts.cf     # Usuarios MTA1
│   ├── postfix-main.cf         # Configuración Postfix MTA1
│   └── postfix-relaymap.cf     # Enrutamiento MTA1
├── config2/                    # Configuración MTA2  
│   ├── postfix-accounts.cf     # Usuarios MTA2
│   ├── postfix-main.cf         # Configuración Postfix MTA2
│   └── postfix-transport.cf    # Enrutamiento MTA2
└── config/                     # Configuración WireGuard
    └── wireguard/
```

## 🔧 Configuraciones importantes

### Volúmenes Docker nombrados
El proyecto usa volúmenes Docker nombrados en lugar de bind mounts para evitar problemas de permisos en Windows:

```yaml
volumes:
  maildata1_vol:
    driver: local
  mailstate1_vol:
    driver: local
```

### Enrutamiento entre dominios
Los servidores están configurados para enrutar automáticamente correos entre dominios:

- **MTA1** → **MTA2**: `example2.local smtp:[mta2]:25`
- **MTA2** → **MTA1**: `example1.local smtp:[mta1]:25`

### Servicios deshabilitados
Para evitar conflictos y problemas de rendimiento:

```yaml
environment:
  - ENABLE_OPENDKIM=0
  - ENABLE_OPENDMARC=0  
  - ENABLE_POLICYD_SPF=0
  - ENABLE_AMAVIS=0
```

## 🐛 Problemas comunes resueltos

1. **"queue file write error"**: Resuelto con volúmenes Docker nombrados
2. **Errores de autenticación SSL**: Configuración TLS corregida  
3. **Correos no llegan**: Sistema de enrutamiento configurado
4. **Problemas de permisos**: Volúmenes Docker en lugar de bind mounts

## 📝 Personalización

### Cambiar contraseñas
Para generar nuevas contraseñas:

```bash
# Generar hash de nueva contraseña
docker exec mta1 doveadm pw -s SHA512-CRYPT -p "tu_nueva_contraseña"

# Actualizar en config1/postfix-accounts.cf y config2/postfix-accounts.cf
```

### Agregar más usuarios
Editar los archivos `postfix-accounts.cf`:

```
nuevo_usuario@example1.local|{SHA512-CRYPT}$hash_generado
```

## 🆘 Troubleshooting

### Verificar estado de servicios
```bash
docker-compose ps
```

### Ver logs
```bash
docker logs mta1 --tail 20
docker logs mta2 --tail 20
```

### Verificar colas de correo
```bash
docker exec mta1 postqueue -p
docker exec mta2 postqueue -p
```

### Forzar procesamiento de colas
```bash
docker exec mta1 postqueue -f
docker exec mta2 postqueue -f
```

## 🌐 Puertos utilizados

| Servicio | Puerto | Descripción |
|----------|---------|-------------|
| VPN WireGuard | 51820/udp | VPN Server |
| Webmail1 | 8081 | Interface web MTA1 |
| Webmail2 | 8082 | Interface web MTA2 |
| MTA1 SMTP | 2525 | SMTP MTA1 (externo) |
| MTA1 IMAP | 2143 | IMAP MTA1 (externo) |
| MTA1 Submission | 2587 | SMTP Submission MTA1 |
| MTA2 SMTP | 3525 | SMTP MTA2 (externo) |
| MTA2 IMAP | 3143 | IMAP MTA2 (externo) |  
| MTA2 Submission | 3587 | SMTP Submission MTA2 |

## 📄 Licencia

MIT License - Ver [LICENSE](LICENSE) para más detalles.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Fork del proyecto
2. Crear branch para nueva funcionalidad
3. Commit de cambios
4. Push al branch
5. Crear Pull Request

## 📧 Soporte

Si encuentras problemas:

1. Revisa la sección de troubleshooting
2. Verifica los logs de los contenedores
3. Crea un issue con información detallada

---

**Desarrollado con ❤️ usando Docker y docker-mailserver**
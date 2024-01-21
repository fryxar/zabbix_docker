Instalacion ZABBIX:

	- Deshabilitar SELinux

	- Instalar Zabbix agent a nivel de Sistema Operativo

		# yum install zabbix-agent

		# Editar el archivo /etc/zabbix_agentd.conf, y editar el siguiente parametro

			Server=172.16.238.0/24

        - Configurar SSL

                # podman volume create zabbix_zabbix-web-ssl

		# cd /var/lib/containers/storage/volumes/zabbix_zabbix-web-ssl/_data/
	
		# cat >openssl.conf <<EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = AR
ST = State
L = City
O = Organization
CN = zabbix.local

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = zabbix.local
EOF

		# openssl req -x509 -nodes -newkey rsa:2048 -keyout ssl.key -out ssl.crt -days 2000 -subj "/C=AR/ST=State/L=City/O=Organization/CN=zabbix.local" -config openssl.conf

		# openssl dhparam -out dhparam.pem 4096

		# chown 1997:1995 /var/lib/containers/storage/volumes/zabbix_zabbix-web-ssl/_data/*

		# cd /root/zabbix

	- Crear secreto

		# echo "secreto" >.password_pgsql.txt

        - Inicial por primera vez los servicios

                # podman-compose up -d

        - Reiniciar los servicios

                # podman-compose down

                # podman-compose up -d

	- Instalar servicio

		# cat <<'EOF' >/etc/systemd/system/podman-compose\@.service
[Unit]
Description=%i (podman-compose)

[Service]
Type=simple
EnvironmentFile=/root/%i/.env_systemd
ExecStartPre=-/usr/bin/podman-compose --in-pod=1 up --no-start
ExecStartPre=/usr/bin/podman pod start pod_%i
ExecStart=/usr/bin/podman-compose wait
ExecStop=/usr/bin/podman pod stop pod_%i

[Install]
WantedBy=default.target
EOF

		# systemctl  enable --now 'podman-compose@zabbix'


	- Actualizar servicio

		# cd /root/zabbix
		# podman-compose pull
		# systemctl  restart 'podman-compose@zabbix'

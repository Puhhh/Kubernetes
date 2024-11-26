## Шаг 1: Генерация CA сертификата
#### Создайте приватный ключ для CA:
```bash
openssl genpkey -algorithm RSA -out ca.key -pkeyopt rsa_keygen_bits:4096
```
#### Создайте самоподписанный сертификат для CA:
```bash
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt \
-subj "/C=RU/ST=Moscow/L=Moscow/O=MyOrganization/OU=IT/CN=MyRootCA"
```
## Шаг 2: Генерация промежуточного сертификата и его подписание CA сертификатом
#### Создайте приватный ключ для промежуточного сертификата:
```bash
openssl genpkey -algorithm RSA -out intermediate.key -pkeyopt rsa_keygen_bits:4096
```
#### Создайте запрос на подписание сертификата (CSR) для промежуточного сертификата:
```bash
openssl req -new -key intermediate.key -out intermediate.csr \
-subj "/C=RU/ST=Moscow/L=Moscow/O=MyOrganization/OU=IT/CN=MyIntermediateCA"
```
#### Подпишите CSR с помощью CA сертификата, чтобы получить промежуточный сертификат:
```bash
openssl x509 -req -in intermediate.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
-out intermediate.crt -days 3650 -sha256
```
## Шаг 3: Генерация сертификата для keycloak.kubernetes.local
#### Создайте приватный ключ для домена keycloak.kubernetes.local:
```bash
openssl genpkey -algorithm RSA -out keycloak.key -pkeyopt rsa_keygen_bits:2048
```
#### Создайте запрос на подписание сертификата (CSR) для домена:
```bash
openssl req -new -key keycloak.key -out keycloak.csr \
-subj "/C=RU/ST=Moscow/L=Moscow/O=MyOrganization/OU=IT/CN=keycloak.kubernetes.local"
```
#### Создайте конфигурационный файл keycloak.ext для расширений, необходимых для SSL сертификатов:
```bash
cat > keycloak.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = keycloak.kubernetes.local
EOF
```
#### Подпишите CSR с помощью промежуточного сертификата, чтобы получить SSL сертификат для домена keycloak.kubernetes.local:
```bash
openssl x509 -req -in keycloak.csr -CA intermediate.crt -CAkey intermediate.key -CAcreateserial \
-out keycloak.crt -days 365 -sha256 -extfile keycloak.ext
```
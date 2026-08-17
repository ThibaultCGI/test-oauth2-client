# test-oauth2-client

Client OpenID Connect (OIDC) de démonstration permettant de s'authentifier auprès de l'application `auth-server` via le flow **Authorization Code + PKCE**.

Une fois authentifié, le client affiche les claims de l'utilisateur connecté contenus dans l'ID Token.

---

## Fonctionnalités

- Authentification OpenID Connect (OIDC)
- Authorization Code Flow
- PKCE activé automatiquement par Spring Security
- Validation automatique de l'ID Token
- Découverte automatique de la configuration OIDC via `issuer-uri`
- Affichage des claims utilisateur

---

## Architecture

```text
+----------------------+
| test-oauth2-client   |
| localhost:8082       |
+----------+-----------+
           |
           | OIDC
           |
           v
+----------------------+
| auth-server          |
| localhost:8080       |
+----------------------+
```

Le client :

1. Redirige l'utilisateur vers l'Authorization Server.
2. L'utilisateur s'authentifie.
3. Le client récupère un Authorization Code.
4. Le client échange ce code contre :
    - un Access Token
    - un ID Token
5. Le client valide l'ID Token grâce aux clés publiques exposées par l'Authorization Server.
6. Les claims de l'utilisateur sont affichés.

---

## Prérequis

- Java 25
- Maven 3.9+
- Application `auth-server` démarrée

---

## Configuration

### application-local.properties

```properties
env.client-id=MyMNQU-awjBR1-p8cIOC-DcjtxB
env.client-secret=xxxxxxxxxxxxxxxx
env.auth-server-url=http://localhost:8080
```

### Configuration du client OIDC

```properties
spring.security.oauth2.client.registration.auth-server.client-id=${client-id}
spring.security.oauth2.client.registration.auth-server.client-secret=${client-secret}

spring.security.oauth2.client.registration.auth-server.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.auth-server.client-authentication-method=client_secret_basic

spring.security.oauth2.client.registration.auth-server.scope[0]=openid
spring.security.oauth2.client.registration.auth-server.scope[1]=trs:produit-api.read

spring.security.oauth2.client.registration.auth-server.redirect-uri={baseUrl}/login/oauth2/code/auth-server

spring.security.oauth2.client.provider.auth-server.issuer-uri=${auth-server.url}
```

---

## Scopes requis

Le client doit être autorisé à demander les scopes suivants :

```text
openid
trs:produit-api.read
```

Le scope `openid` active les fonctionnalités OpenID Connect.

---

## Endpoints utilisés

Grâce à la propriété :

```properties
spring.security.oauth2.client.provider.auth-server.issuer-uri=http://localhost:8080
```

Spring découvre automatiquement les endpoints OIDC exposés par l'Authorization Server :

```text
/.well-known/openid-configuration
/oauth2/authorize
/oauth2/token
/oauth2/jwks
/userinfo
```

---

## Séquence d'authentification

```text
Utilisateur
    |
    v
test-oauth2-client
    |
    | GET /oauth2/authorization/auth-server
    v
auth-server
    |
    | Authentification utilisateur
    v
Authorization Code
    |
    v
test-oauth2-client
    |
    | Echange du code contre des tokens
    v
ID Token + Access Token
    |
    v
Utilisateur authentifié
```

---

## Lancement

### Démarrer l'Authorization Server

```bash
mvn spring-boot:run
```

### Démarrer le client

```bash
mvn spring-boot:run
```

### Accéder à l'application

```text
http://localhost:8082
```

---

## Exemple de résultat

Après authentification :

```json
{
  "sub": "admin",
  "iss": "http://localhost:8080",
  "aud": [
    "MyMNQU-awjBR1-p8cIOC-DcjtxB"
  ],
  "application_code": "TOC",
  "client_id": "MyMNQU-awjBR1-p8cIOC-DcjtxB"
}
```

---

## Sécurité

Le client utilise :

- Authorization Code Flow
- PKCE
- Validation de signature des JWT via JWKS
- Validation du nonce OIDC
- Sessions Spring Security

---

## Technologies

- Java 25
- Spring Boot 4.1
- Spring Security 7.1
- OpenID Connect 1.0
- OAuth 2.1

---

## Evolutions possibles

- Appel d'un Resource Server protégé par JWT
- Interface utilisateur personnalisée
- Gestion du logout OIDC
- Refresh Token
- Gestion des rôles et permissions
- Consommation de l'endpoint UserInfo

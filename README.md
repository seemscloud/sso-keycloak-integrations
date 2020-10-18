# SSO with `Keycloak`

## Realm Management

### Export Realm with All Confidentials
```bash
/opt/jboss/keycloak/bin/standalone.sh \
  -Djboss.socket.binding.port-offset=100 \
  -Dkeycloak.migration.action=export \
  -Dkeycloak.migration.provider=singleFile \
  -Dkeycloak.migration.realmName=seems-cloud \
  -Dkeycloak.migration.usersExportStrategy=REALM_FILE \
  -Dkeycloak.migration.file=/tmp/realm.json
```

### Import Realm with All Confidentials
```bash
/opt/jboss/keycloak/bin/standalone.sh \
  -Djboss.socket.binding.port-offset=100 \
  -Dkeycloak.migration.action=import \
  -Dkeycloak.migration.provider=singleFile \
  -Dkeycloak.migration.strategy=OVERWRITE_EXISTING \
  -Dkeycloak.migration.file=/tmp/realm.json
```

### Authentication
```bash
/bin/bash /opt/jboss/keycloak/bin/kcadm.sh config credentials \
    --server http://localhost:8080/auth \
    --realm master \
    --user seems \
    --password seems.cloud
```

### Create Realm
```bash
/bin/bash /opt/jboss/keycloak/bin/kcadm.sh create realms \
    --set realm=seems-cloud \
    --set enabled=true \
    --set displayName="Seem Cloud" \
    --set displayNameHtml="Seems Cloud"
```

### Delete Realm
```bash
/bin/bash /opt/jboss/keycloak/bin/kcadm.sh delete \
    --target-realm=seems-cloud \
    realms/seems-cloud
```

## SSO Services

### Grafana Configuration Snippet `grafana.ini`
```bash
[server]
root_url = http://grafana.seems.cloud

[auth.generic_oauth]
enabled = true
name = Keycloak
allow_sign_up = true
client_id = grafana.seems.cloud
client_secret = 228ce970-742e-4b72-bf5f-e208776b890c
scopes = openid profile email
email_attribute_name = email:primary
;email_attribute_path =
auth_url = https://keycloak.seems.cloud/auth/realms/seems-cloud/protocol/openid-connect/auth
token_url = https://keycloak.seems.cloud/auth/realms/seems-cloud/protocol/openid-connect/token
api_url = https://keycloak.seems.cloud/auth/realms/seems-cloud/protocol/openid-connect/userinfo
;allowed_domains =
;team_ids =
;allowed_organizations =
;role_attribute_path =
tls_skip_verify_insecure = true
;tls_client_cert =
;tls_client_key =
;tls_client_ca =

[auth.proxy]
enabled = true
```

### SonarQube Configuration Snippet `sonar.properties`
```bash
sonar.forceAuthentication=true
sonar.auth.saml.enabled=true
sonar.auth.saml.applicationId=sonarqube.seems.cloud
sonar.auth.saml.providerName=Keycloak
sonar.auth.saml.providerId=https://keycloak.seems.cloud/auth/realms/seems-cloud 
sonar.auth.saml.loginUrl=https://keycloak.seems.cloud/auth/realms/seems-cloud/protocol/saml/clients/sonarqube.seems.cloud 
sonar.auth.saml.certificate.secured=REALM_CERTIFICATE
sonar.auth.saml.user.login=login
sonar.auth.saml.user.name=name
sonar.auth.saml.user.email=email
sonar.auth.saml.group.name=groups
```

### Gitlab Configuration Snippet `gitlab.rb`
```bash
gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = ['saml']
gitlab_rails['omniauth_block_auto_created_users'] = false
gitlab_rails['omniauth_auto_link_saml_user'] = true
gitlab_rails['omniauth_providers'] = [
  {
    name: 'saml',
    args: {
             assertion_consumer_service_url: 'https://gitlab.seems.cloud/users/auth/saml/callback',
             idp_cert_fingerprint: 'A4:F1:5D:EF:E5:7E:F7:3E:78:59:59:09:96:7B:8D:11:F7:A8:0B:45',
             idp_sso_target_url: 'https://keycloak.seems.cloud/auth/realms/seems-cloud/protocol/saml/clients/gitlab.seems.cloud',
             issuer: '1git0.localdomain',
             attribute_statements: {
                     first_name: ['first_name'],
                     last_name: ['last_name'],
                     name: ['name'],
                     username: ['name'],
                     email: ['email']
             },
             name_identifier_format: 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent'
           },
    label: 'Seems Cloud'
  }
]
```
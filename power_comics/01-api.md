# Power Comics API

## O que abordaremos?
Com o intuito de apenas passar por alguns passos importantes e que podem ajudar outras pessoas a iniciarem o desenvolvimento de suas aplicações symfony, vamos documentar apenas ALGUNS passos, não todo o projeto em si.

### Ambiente
Antes de mais nada, famos instalar o 'básico':
1. Instalação do Symfony
```shell
composer create-project symfony/skeleton:"6.4.*" my_project_directory
```
2. Instalação do API Platform
```shell
composer require api
```

### Autênticação JWT
Tá aí uma coisa um pouco mais "complicada" e que levou mais tempo do que eu esperava para realizar todos os passos. Então vamos a eles:
1. Para autênticar, é necessário alguma forma de validar usuário e senha, então usaremos o armazenamento do usuário através do banco de dados, então para usar o 'make' do symfony, vamos instalar o seu bundle.
```shell
composer require symfony/maker-bundle --dev
```
2. Agora vamos usar o make para criar o usuário, definindo qual atributo será usado como unico, como email ou nome de usuário.

![](./data/img001.png)

3. Vamos então criar a tabela no banco de dados, usando a criação do migration e executando-a em seguida.

![](./data/img002.png)

4. Hora de instalar o pacote responsável pelo uso do JWT
```shell
composer require lexik/jwt-authentication-bundle
```

5. Este script em Shell é utilizado para configurar um ambiente de autenticação JWT (JSON Web Token) em um sistema baseado em Linux. Ele gera chaves privadas e públicas para JWT e define permissões de acesso.
```shell
mkdir -p config/jwt
jwt_passphrase=${JWT_PASSPHRASE:-$(grep ''^JWT_PASSPHRASE='' .env | cut -f 2 -d ''='')}
echo "$jwt_passphrase" | openssl genpkey -out config/jwt/private.pem -pass stdin -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
echo "$jwt_passphrase" | openssl pkey -in config/jwt/private.pem -passin stdin -out config/jwt/public.pem -pubout
setfacl -R -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
setfacl -dR -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
```

6. Defina a rota que utilizaremos para validar o login em 'application/config/routes.yaml' adicionando:
```yaml
api_login_check:
    path: /api/login_check
```

7. Agora vamos definir as regras de segurança em 'application/config/packages/security.yaml'
- Adicionado em firewalls o item 'docs' para acesso a documentação da API sem necessidade de login (security: false).
```yaml
firewalls:
    docs:
        pattern: ^/api/docs
        security: false
```
- Sobrescrevendo o item main
   - Definindo o stateless como true para que não manter o estado da sessão entre requisições.
   - Configurando o login por JSON em json_login.
      - check_path: Define o endpoint onde as credenciais de login devem ser enviadas para autenticação.
      - success_handler: Define o manipulador que lida com autenticação bem-sucedida. lexik_jwt_authentication.handler.authentication_success é um serviço que gera e retorna um token JWT.
      - failure_handler: Define o manipulador que lida com falhas de autenticação. lexik_jwt_authentication.handler.authentication_failure é um serviço que retorna uma resposta apropriada em caso de falha de login.
      - jwt: Configura o firewall para usar autenticação JWT. O ~ indica que as configurações padrão do JWT são usadas.
```yaml
main:
    pattern: ^/api
    stateless: true
    json_login:
      check_path: /api/login_check
      success_handler: lexik_jwt_authentication.handler.authentication_success
      failure_handler: lexik_jwt_authentication.handler.authentication_failure
    jwt: ~
```
- Por último, inserir no access_control, os papéis/permissões necessárias para acessar determinadas URLs.
   - IS_AUTHENTICATED_ANONYMOUSLY: Permite acesso a qualquer usuário, mesmo que não esteja autenticado. Necessário para que usuários possam realizar login, acessar a doc e também o endpoint de registro de usuário.
   - IS_AUTHENTICATED_FULLY: Exige que o usuário esteja totalmente autenticado para acessar essas rotas. Isso significa que o usuário deve ter feito login e ter um token de autenticação válido.
   - PUBLIC_ACCESS: É uma constante que permite acesso público a essa rota, significando que qualquer usuário, mesmo que não esteja autenticado, pode acessar essa rota. É semelhante a IS_AUTHENTICATED_ANONYMOUSLY, mas mais explícito no sentido de que não há qualquer requisito de autenticação.
```yaml
access_control:
  - { path: ^/api/register, roles: PUBLIC_ACCESS }
  - { path: ^/api/docs, roles: IS_AUTHENTICATED_ANONYMOUSLY }
  - { path: ^/api/login_check, roles: IS_AUTHENTICATED_ANONYMOUSLY }
  - { path: ^/, roles: IS_AUTHENTICATED_FULLY }
```
8. Vamos criar a controller de registro de usuário para que este usuário possa posteriormente ser autênticado.
- Criando a controller:
```shell
bin/console make:controller RegistrationController
```
- Definindo o conteúdo:
```php
#[Route('/api', name: 'api_')]
class RegistrationController extends AbstractController
{
    #[Route('/register', name: 'register', methods: 'post')]
    public function index(ManagerRegistry $doctrine, Request $request, UserPasswordHasherInterface $passwordHasher): JsonResponse
    {
        $em = $doctrine->getManager();
        $decoded = json_decode($request->getContent());
        $plaintextPassword = $decoded->password;

        $user = new Usuario();
        $hashedPassword = $passwordHasher->hashPassword(
            $user,
            $plaintextPassword
        );
        $user
            ->setPassword($hashedPassword)
            ->setUsername($decoded->username)
            ->setRoles(['ROLE_USER'])
        ;
        $em->persist($user);
        $em->flush();

        return $this->json(['message' => 'Registered Successfully']);
    }
}
```
Obs.: Usamos o 'UserPasswordHasherInterface' para fazer o encoding da senha enviada.
9. Testando a autênticação:

- Cadastro de um usuário:

![](./data/img003.png)

- Obtendo token do usuário cadastrado:

![](./data/img004.png)

- Obtendo dados utilizando token na requisição:

![](./data/img005.png)

#### Todos comandos mencionados até o momento:
```shell
composer create-project symfony/skeleton:"6.4.*" my_project_directory
composer require api
composer require symfony/maker-bundle --dev
bin/console make:user
bin/console make:migration
bin/console doctrine:migrations:migrate
composer require lexik/jwt-authentication-bundle
mkdir -p config/jwt
jwt_passphrase=${JWT_PASSPHRASE:-$(grep ''^JWT_PASSPHRASE='' .env | cut -f 2 -d ''='')}
echo "$jwt_passphrase" | openssl genpkey -out config/jwt/private.pem -pass stdin -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
echo "$jwt_passphrase" | openssl pkey -in config/jwt/private.pem -passin stdin -out config/jwt/public.pem -pubout
setfacl -R -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
setfacl -dR -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
bin/console make:controller RegistrationController
```

auth
```
composer create-project symfony/skeleton:"6.4.*" my_project_directory
composer require api
composer require symfony/maker-bundle --dev
bin/console make:user
bin/console make:migration
bin/console doctrine:migrations:migrate
composer require lexik/jwt-authentication-bundle
mkdir -p config/jwt
jwt_passphrase=${JWT_PASSPHRASE:-$(grep ''^JWT_PASSPHRASE='' .env | cut -f 2 -d ''='')}
echo "$jwt_passphrase" | openssl genpkey -out config/jwt/private.pem -pass stdin -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
echo "$jwt_passphrase" | openssl pkey -in config/jwt/private.pem -passin stdin -out config/jwt/public.pem -pubout
setfacl -R -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
setfacl -dR -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
bin/console make:controller RegistrationController


9  bin/console make:entity
10  bin/console make:migration
11  bin/console doctrine:migrations:migrate
12  bin/console lexik:jwt:generate-keypair
13  composer require lexik/jwt-authentication-bundle
14  chmod +x ./bin/jwt-install
15  ./bin/jwt-install
16  bin\console make:controller RegistrationController
17  bin/console make:controller RegistrationController
18  bin/console make:controller DashboardController
19  bin/console
20  bin/console make:entity
21  bin/console make:migration
22  bin/console doctrine:migrations:migrate
```


security
```yaml
security:
#    role_hierarchy:
#        ROLE_CUSTOMER: ROLE_USER
#        ROLE_COLLABORATOR: ROLE_USER
#        ROLE_ADMIN: [ ROLE_COLLABORATOR, ROLE_CUSTOMER ]
#        ROLE_SUPER_ADMIN: [ ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH ]
    # https://symfony.com/doc/current/security.html#registering-the-user-hashing-passwords
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'
    # https://symfony.com/doc/current/security.html#loading-the-user-the-user-provider
    providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\Usuario
                property: username
    firewalls:
        docs:
            pattern: ^/api/docs
            security: false
        main:
            pattern: ^/(api|login)
            stateless: true
            json_login:
                check_path: /api/login_check
                success_handler: lexik_jwt_authentication.handler.authentication_success
                failure_handler: lexik_jwt_authentication.handler.authentication_failure
            jwt: ~
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

#        main:
#            lazy: true
#            provider: app_user_provider

            # activate different ways to authenticate
            # https://symfony.com/doc/current/security.html#the-firewall

            # https://symfony.com/doc/current/security/impersonating_user.html
            # switch_user: true
#        api:
#            pattern: ^/api
#            stateless: true
#            jwt: ~

    # Easy way to control access for large sections of your site
    # Note: Only the *first* access control that matches will be used
    access_control:
        # Allows accessing the Swagger UI
        - { path: ^/api/docs, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/api/login_check, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/api/register, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        # require ROLE_ADMIN for /admin*
        #        - { path: '^/admin', roles: ROLE_ADMIN }
        # or require ROLE_ADMIN or IS_AUTHENTICATED_FULLY for /admin*
        #        - { path: '^/admin', roles: [ IS_AUTHENTICATED_FULLY, ROLE_ADMIN ] }
        # the 'path' value can be any valid regular expression
        # (this one will match URLs like /api/post/7298 and /api/comment/528491)
        - { path: ^/, roles: IS_AUTHENTICATED_FULLY }

when@test:
    security:
        password_hashers:
            # By default, password hashers are resource intensive and take time. This is
            # important to generate secure password hashes. In tests however, secure hashes
            # are not important, waste resources and increase test times. The following
            # reduces the work factor to the lowest possible values.
            Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface:
                algorithm: auto
                cost: 15 # Lowest possible value for bcrypt
                time_cost: 3 # Lowest possible value for argon
                memory_cost: 10 # Lowest possible value for argon

```
routes
```yaml
controllers:
    resource:
        path: ../src/Controller/
        namespace: App\Controller
    type: attribute
api_login_check:
    path: /api/login_check

```

registration controller
```php
<?php

namespace App\Controller;

use App\Entity\Usuario;
use Doctrine\Persistence\ManagerRegistry;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/api', name: 'api_')]
class RegistrationController extends AbstractController
{
    #[Route('/register', name: 'register', methods: 'post')]
    public function index(ManagerRegistry $doctrine, Request $request, UserPasswordHasherInterface $passwordHasher): JsonResponse
    {
        $em = $doctrine->getManager();
        $decoded = json_decode($request->getContent());
        $plaintextPassword = $decoded->password;

        $user = new Usuario();
        $hashedPassword = $passwordHasher->hashPassword(
            $user,
            $plaintextPassword
        );
        $user
            ->setPassword($hashedPassword)
            ->setStatus(1)
            ->setUsername($decoded->username)
            ->setDataCadastro(new \DateTime())
            ->setRoles(['ROLE_USER'])
        ;
        $em->persist($user);
        $em->flush();

        return $this->json(['message' => 'Registered Successfully']);
    }
}

```

jwt-install
```shell
#!/usr/bin/env sh
set -e
    apt-get update --yes
    apt-get install acl --yes
    mkdir -p config/jwt
    jwt_passphrase=${JWT_PASSPHRASE:-$(grep ''^JWT_PASSPHRASE='' .env | cut -f 2 -d ''='')}
    echo "$jwt_passphrase" | openssl genpkey -out config/jwt/private.pem -pass stdin -aes256 -algorithm rsa -pkeyopt rsa_keygen_bits:4096
    echo "$jwt_passphrase" | openssl pkey -in config/jwt/private.pem -passin stdin -out config/jwt/public.pem -pubout
    setfacl -R -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt
    setfacl -dR -m u:www-data:rX -m u:"$(whoami)":rwX config/jwt

```
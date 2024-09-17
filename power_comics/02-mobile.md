# Power Comics Mobile

## O que abordaremos?
Assim como no desenvolvimento da API, apresentaremos apenas os passos principais para que as pessoas possam iniciar o desenvolvimento de suas aplicações flutter.

## Ambiente e target android
Antes de mais nada, vamos [iniciar o projeto](https://docs.flutter.dev/reference/flutter-cli).
```shell
flutter create --project-name powercomics --platforms android --org br.com.powercomics ./power-comics
```
Como o aplicativo (obviamente) utilizará dados vindos da internet, iremos [configurar o Android manifest](https://developer.android.com/develop/connectivity/network-ops/connecting) para que o acesso seja liberado.
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <application>
        ... configurações ..
    </application>
    <queries>
        ... configurações ..
    </queries>
</manifest>
```
Variáveis de ambiente são suportadas nativamente pelo flutter a partir da versão 3.7, cokm isso a vida ficou mais bela rsrsrsrsrs

Para definir essas variáveis e especificar quais usar em cada momento (desenvolvimento e produção por exemplo), criei na raiz do projeto a pasta env, e nela criei o dev.json e também o prod.json, ambas seguindo a seguinte estrutura:
```json
{
    "ENDPOINT": ""
}
```
Agora eu tenho meu arquivo que armazenará minhas variáveis de ambiente, não serão versionadas, e serão usadas em tempo de compilação, fazendo com que estejam guardadas de forma mais segura dentro do app.

Para executar o app com essas variáveis, será necessário (no vscode), seguir os sequintes passos:
- Vá no item "Executar e Depurar" (menu com um bug e um play)
- Clique na opção "...crie um arquivo launch.json"
- No menu apresentado no centro da parte superior da tela, clique em "Dart & Flutter"
- Será gerado o .vscode/launch.json, então dentro do 'configurations', o primeiro objeto (o que está sem profile mode ou release mode), crie a estrutura abaixo (após o 'type'):
```json
"args": [
    "--dart-define-from-file",
    "env/dev.json"
]
```

Ficando desta forma:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "power-comics",
            "request": "launch",
            "type": "dart",
            "args": [
                "--dart-define-from-file",
                "env/dev.json"
            ]
        },
        {
            "name": "power-comics (profile mode)",
            ... configurações ...
        },
        {
            "name": "power-comics (release mode)",
            ... configurações ...
        }
    ]
}
```
O último item que gostaria de deixar pronto antes de começar a codar de fato, é a definição das versões minimas e versão algo para as quais o app será desenvolvido (android), então, vamos configurar o 'android/app/build.gradle' para usar o local.properties, que é gerado automáticamente ao fazer o build do app, e fica localizado na raiz da pasta android.

Como inicialmente pretendo dar suporte ao android 9 até o (por enquanto) vindouro android 15, consultando a [lista de APIs x versão do android](https://developer.android.com/tools/releases/platforms), vi que o android 9 é a API 28 e o android 15 é a API 35, criei então no meu android/local.properties os seguintes itens:
```text
flutter.minSdkVersion=28
flutter.targetSdkVersion=35
```

Para usar esses dados, lá no nosso 'build.gradle', adicionei as variáveis vindas do local properties, logo após a definição de plugins, definindo também minSdk e o targetSdk do defaultConfig para usar os dados vindos destas variáveis, deixando o arquivo mais ou menos assim:
```gradle
plugins {
  ... configurações ...
}

def localProperties = new Properties()
def localPropertiesFile = rootProject.file('local.properties')
if (localPropertiesFile.exists()) {
    localPropertiesFile.withReader('UTF-8') { reader ->
        localProperties.load(reader)
    }
}

def flutterMinSdkVersion = localProperties.getProperty('flutter.minSdkVersion')
if (flutterMinSdkVersion == null) {
    flutterMinSdkVersion = flutter.minSdkVersion.toString();
}

def flutterTargetSdkVersion = localProperties.getProperty('flutter.targetSdkVersion')
if (flutterTargetSdkVersion == null) {
    flutterTargetSdkVersion = flutter.targetSdkVersion.toString();
}

android {
    ... configurações ...

    defaultConfig {
        ... configurações ...
        minSdk = flutterMinSdkVersion.toInteger()
        targetSdk = flutterTargetSdkVersion.toInteger()
        ... configurações ...
    }

    buildTypes {
        ... configurações ...
    }
}

flutter {
    source = "../.."
}
```
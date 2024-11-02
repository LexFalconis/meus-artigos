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

## Dependências iniciais

Uma coisa que eu já vim decidido é que `não quero usar o getx neste projeto`, e os motivos principais são:
1. É o único gerenciador de estado que posso dizer realmente que usei e me sinto na necessidade de usar outros.
2. Este incrível package está a mais de 1 ano sem novas publicações no [pub.dev](https://pub.dev/packages/get)

Então, o que tenho de alternativas?

Bom, tenho o [mobx](https://pub.dev/packages/mobx) que me parece simples e funcional, mas que também está alguns meses (6 p ser mais exato) sem atualização, então vou evitar.

Temos o [bloc](https://pub.dev/packages/flutter_bloc), mas confesso que ele me pareceu em alguns pontos parecido com o getx em seu uso, e minha idéia no momento é estudar algo TALVEZ um pouco mais diferente para sair da zona de conforto.

Queria ver o [provider](https://pub.dev/packages/provider) já tem um tempinho, mas sei que ele está sendo substituido pelo [riverpod](https://pub.dev/packages/riverpod), que é sua "nova versão", mas com tantas alterações que a equipe por trás dele acho mais interessante criar um novo package para substituir o anterior, sem a preocupação dos usuários de ter uma quebra extremamente grande de compatibilidade ao mudar sua major.

Então é isso, vou de riverpod.

Com esta definição feita, vamos adicionar ele no nosso pubspec.yaml, adicionei a então o `flutter_riverpod: ^2.5.3`.

Outro ponto, que não necessáriamente precisava ser feito agora, é inserir o package que cuidará das requisições HTTP, para este requisito, o [dio](https://pub.dev/packages/dio) foi o escolhido pelo seu poder e simplicidade no uso.

## Path das imagens

Para utilizar imagens na aplicação, vamos centralizar o local onde elas ficam, para facilitar a gestão desses arquivos, então foi necessário inserir também no pubspec, o path das imagens ficando assim:

```yaml
  assets:
    - assets/images/
```

## Tela de carregamento e o main.dart iniciais

Como o Flutter precisa estar totalmente carregado para que possamos definir a orientação do app (que funcionará apenas na vertical), usaremos o 'WidgetsFlutterBinding.ensureInitialized()' para garantir que isso ocorra.

O próximo passo será exatamente definir a orientação usando o 'SystemChrome.setPreferredOrientations', e por fim, vamos iniciar o aplicativo, mas encapsulando-o dentro do ProviderScope, que irá gerar um escopo global para nossa gestão de dependências.

Como ficou nosso main:

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
  ]).then((_) {
    runApp(
      const ProviderScope(
        child: MyApp(),
      ),
    );
  });
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Power Comics',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      initialRoute: PagesRoutes.splashScreen,
      onGenerateRoute: AppRouter.onGenerateRoute,
      home: const SplashScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

O MyApp foi criado como um StatelessWidget, pois ao menos por enquanto, não haverá mudanças dinâmicas dentro deste widget. Ele retorna o MaterialApp, onde defini que a rota inicial do aplicativo é a SplashScreen, e o onGenerateRoute define a função personalizada para geração de rotas.

Criei o AppRouter para ser o responsável pelas nossas rotas, e é onde está a lógica do onGenerateRoute.

```dart
class AppRouter {
  static Route<dynamic> onGenerateRoute(RouteSettings settings) {
    switch (settings.name) {
      case PagesRoutes.splashScreen:
        return MaterialPageRoute(builder: (_) => const SplashScreen());
      case PagesRoutes.home:
        return MaterialPageRoute(builder: (_) => const MyHomePage());
      default:
        return MaterialPageRoute(
          builder: (_) => Scaffold(
            body: Center(
              child: Text('No route defined for ${settings.name}'),
            ),
          ),
        );
    }
  }
}

abstract class PagesRoutes {
  static const String root = '/';
  static const String splashScreen = '/splash-screen';
  static const String home = '/home';
}
```

Já a tela de carregamento, possui em seu initstate, a funcionalidade de redirecionamento de tela após um segundo e mail, enquanto isso, ela exibe uma imagem e o texto de carregamento.

```dart
class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    Future.delayed(const Duration(milliseconds: 1500), () {
      Navigator.of(context).pushReplacementNamed(PagesRoutes.home);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Material(
      color: const Color.fromRGBO(76, 76, 76, 1),
      child: Container(
        alignment: Alignment.center,
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Image.asset(
              PathImages.splashscreen,
              fit: BoxFit.cover,
            ),
            const SizedBox(
              height: 10,
            ),
            const Text(
              'Carregando...',
              style: TextStyle(color: Colors.white, fontSize: 25,),
            ),
          ],
        ),
      ),
    );
  }
}

```

## Criação do menu

Para facilitar a vida, é bom ter um menu com as opções que o usuário possui, sendo apresentado de forma padrão em todas as telas, certo? 
Então suas páginas precisam retornar um Scaffold com um menu no atributo drawer. 
O menu não é tão complicado de ser criada uma view.dart com um StatefulWidget, onde seu retorno é o widget PopScope.

O [PopScope](https://api.flutter.dev/flutter/widgets/PopScope-class.html) permitirá gerênciar os gestos de navegação, ocultando o menu ao clicar fora dele ou pressionando o botão voltar do celular.

Bem, então em seu child vamos inserir um Drawer com um ListView, para ficar com uma aparência legal de menu, e também uma imagem/logo antes das opções do menu, deixando sua versão simplificada mais ou menos assim:

```dart
class MenuView extends StatefulWidget {
  const MenuView({super.key});

  @override
  State<MenuView> createState() => _MenuViewState();
}

class _MenuViewState extends State<MenuView> {
  PackageInfo _packageInfo = PackageInfo(
    appName: '',
    packageName: '',
    version: '',
    buildNumber: '',
    buildSignature: '',
    installerStore: '',
  );

  @override
  void initState() {
    super.initState();
    _initPackageInfo();
  }

  Future<void> _initPackageInfo() async {
    final info = await PackageInfo.fromPlatform();
    setState(() {
      _packageInfo = info;
    });
  }
  @override
  Widget build(BuildContext context) {
    return PopScope(
      canPop: false,
      onPopInvokedWithResult: (bool didPop, Object? result) async {
        if (didPop) {
          return;
        }
        Navigator.pop(context);
      },
      child: Drawer(
        backgroundColor: CustomColors.customDarkGreyColor,
        child: Column(
          children: [
            Expanded(
              child: ListView(
                padding: EdgeInsets.zero,
                children: [
                  ///////////////////////////////////////
                  // Topo do menu. Logo
                  ///////////////////////////////////////
                  Column(
                    children: [
                      const SizedBox(
                        height: 15,
                      ),
                      Padding(
                        padding: const EdgeInsets.all(8.0),
                        child: Image.asset(
                          PathImages.logo,
                          height: 150,
                        ),
                      ),
                    ],
                  ),

                  const Divider(),

                  ///////////////////////////////////////
                  // Inicio das opções do menu
                  ///////////////////////////////////////

                  // Botão Inicio/home
                  ListTile(
                      minTileHeight: 20,
                      leading: Image.asset(
                        imagePath,
                        height: 40,
                      ),
                      title: Text(
                        texto,
                        style: CustomTextStyle.bodyStyle(color: cor),
                      ),
                      subtitle: Text(
                        descricao,
                        style: CustomTextStyle.secondaryTextStyle(color: cor),
                      ),
                      onTap: () {
                        // if (authController.currentPage == PagesRoutes.homeRoute) {
                        //   Navigator.pop(context);
                        //   return;
                        // }
                
                        // Get.offAllNamed(PagesRoutes.homeRoute);
                      },
                    );
                ],
              ),
            ),

            // VERSÃO DO APP
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 20.0, vertical: 18.0),
              child: Row(
                children: [
                  const Icon(Icons.info_outline, size: 20),
                  Text(' v${_packageInfo.version}+${_packageInfo.buildNumber}'),
                ],
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

Você deve ter notado o uso do [PackageInfo](https://pub.dev/packages/package_info_plus), bom, este package irá permitir 
que acrescentemos no menu, a versão atual do app, com base no 'version' do nosso pubspec. 

Ah, um detalhe que é importante mencionar, caso adicione este package em seu projeto, não se assuste se ao adicionar a 
tela que usa ele quebrar rsrsrsrs, não adianta fazer hot reload ou restart do app, será necessário parar a execução e buildar novamente a aplicação.


- Criado menu
- Criado form com autenticação
- Adicionei (json_annotation, json_serializable), freezed, freezed_annotation e build_runner
- usei o json_serializable e json_annotation no model para a serialização e depois o build para gerar o arquivo 'part' 
- executei `dart run build_runner build`
- criado o AuthResult e usado nele o freezed e o freezed_annotation
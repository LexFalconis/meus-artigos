# Publicação na loja do google 

## O que abordaremos?
Os principais pontos para a publicação do app, desde a geração da chave de autenticação necessária para a publicação, 
publicação do app para teste fechado (necessário por ao menos 14 dias para contas de pessoa física) até a publicação final na loja do Google.

## Preparações finais do app

A primeira coisa que quero documentar, é a adição do logo do projeto como icone no aplicativo.

Com a imagem que será usada "em mãos", vamos para o [icon kitchen](https://icon.kitchen/), ele nos ajudará a criar cada 
imagem que precisamos (muitos tamanhos diferentes), facilitando bastante a vida.

No meu caso, vou clicar na opção "Image", fazser upload da imagem, reduzir o padding padrão de 15 para 5% e definir o background como branco.

![img 001](data/publicacao-loja/001.png)

Simples né? Agora basta fazer o download, descompactar, e então partiremos para a IDE de sua escolha.

Ao abrir o projeto, vá em "android > app > src > main > res", e substitua as pastas "mipmap-*" pelas que baixou no passo anterior.

![img 002](data/publicacao-loja/002.png)

Agora vamos abrir o "android > app > src > main > AndroidManifest.xml", alterar a opção (caso necessário) "android:label", 
é lá que alteramos o nome que será apresentado no celular como nome do app.

## Criação e utilização da chave de publicação

Seguindo a documentação do Flutter, temos a [geração de keystore](https://docs.flutter.dev/deployment/android#signing-the-app) para upload 
do aplicativo.

Basicamente, executamos o seguinte comando no terminal (considerando que estou usando Linux).

```shell
keytool -genkey -v -keystore ~/my-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
```
Alterando conforme suas necessidades. O meu, ficou assim:

![img 003](data/publicacao-loja/003.png)

Agora guarde em um local seguro, pois precisaremos dela sempre que for enviar o app para a loja.

Em seguida, faça uma cópia para "android > app", e crie o arquivo com as configurações da chave na raiz do android, 
ficando como "android/key.properties", e seu conteúdo sendo algo similar a isso:
```
storePassword=senhaQueColocouNaCriaçãoDaChave
keyPassword=senhaQueColocouNaCriaçãoDaChave
keyAlias=aliasDefinidoNaCriaçãoDaChave
storeFile=nome-do-arquivo.jks
```

![img 004](data/publicacao-loja/004.png)

Só para constar, no parâmetro de criação da chave onde colocamos a validade como 10000 dias, não foi de alegre não ok? rsrsrsrs o [Google recomenda que a chave seja válida por ao menos 25 anos.](https://developer.android.com/studio/publish/app-signing?hl=pt-br#generate-key)

Depois da chave Criada e armazenada em seu devido lugar, vamos preparar o gradle para usa-la.

Em "androoid > app > build.gradle", vamos inserir a configuração de propriedade para nossa keystore.

Colocaremos as definições do keystore antes do "android {}"
```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if(keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
```

Já as configurações, colocaremos dentro do "android {}", após o defaultConfig e antes do buildTypes.

```gradle
signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
```

E a última alteração neste arquivo será mudar o "signingConfig = signingConfigs.debug" para 
"signingConfig = signingConfigs.release", dentro do buildTypes.

![img 005](data/publicacao-loja/005.png)

### Hora do build

No terminal, vamos executar o comando para o build do projeto.

```shell
flutter build appbundle --dart-define-from-file=./env/prod.json
```

Estou usando o "--dart-define-from-file" para definir o arquivo onde estão as variáveis de ambiente.

### Preenchimento dos dados na loja do Google

Acessando o [Google Play Console](https://play.google.com/console/u/0/signup) com a conta criada anteriormente.

Ao acessar, temos já a sugestão para "criar o primeiro app", vamos então nesta opção.

![006.png](data/publicacao-loja/006.png)

A primeira parte do formulário é muito legal, ela faz parecer que é pouca coisa e que tudo é simples, mas não se engane rsrsrsrsrs

![007.png](data/publicacao-loja/007.png)

Temos bastante coisa para preencher, mas a tela seguinte já nos informa que vamos precisar obrigatoriamente realizar um 
teste fechado com 12 usuários por 14 dias antes de poder enviar o app para produção. 

![008.png](data/publicacao-loja/008.png)

#### Informar nossa equipe sobre o conteúdo do app
Vamos começar com a configuração do App.

![009.png](data/publicacao-loja/009.png)

O Primeiro item foi a politica de privacidade. Como Já sabia que isso era necessário, um tempo atrás havia "explicado" para 
algumas IAs tudo que fazemos e precisamos de dados, então fui procurando em cada "resposta" quais eram os pontos que de fato 
atendem as nossas necessidades, então coloquei no Google Docs, e aqui coloquei o link.

![010.png](data/publicacao-loja/010.png)

Agora definimos os acessos ao aplicativo, pois com as devidas instruções, usuário e senha informados, eles podem testar melhor 
o app para aprovar ou não a publicação.

![011.png](data/publicacao-loja/011.png)

![012.png](data/publicacao-loja/012.png)

Definimos os anúncios, mas aqui não tenho muito o que preencher, pois não exibirei anúncios.

![013.png](data/publicacao-loja/013.png)

Agora vamos definir a classificação de conteúdo.

![014.png](data/publicacao-loja/014.png)

![015.png](data/publicacao-loja/015.png)

![016.png](data/publicacao-loja/016.png)

![017.png](data/publicacao-loja/017.png)

Já sobre o público alvo, como a ideia não é atender especificamente crianças e entender que os usuários são maiores de 
idade (ao menos a grande maioria), vou colocar que o público alvo são maiores de 18 anos.

![018.png](data/publicacao-loja/018.png)

![019.png](data/publicacao-loja/019.png)

Não estamos lançando um app de notícias, então é mais um ponto que não temos muito o que declarar.

![020.png](data/publicacao-loja/020.png)

No item segurança de dados, temos que declarar o que usamos/armazenamos de dados dos usuários.

![021.png](data/publicacao-loja/021.png)

![022.png](data/publicacao-loja/022.png)

![023.png](data/publicacao-loja/023.png)

![024.png](data/publicacao-loja/024.png)

![025.png](data/publicacao-loja/025.png)

![026.png](data/publicacao-loja/026.png)

![027.png](data/publicacao-loja/027.png)

Com base no que foi respondido, novas questões serão apresentadas no passo seguinte.

![028.png](data/publicacao-loja/028.png)

![029.png](data/publicacao-loja/029.png)

![030.png](data/publicacao-loja/030.png)

![031.png](data/publicacao-loja/031.png)

Eles questionam também se é um app governamental, adivinha a nossa resposta?

![032.png](data/publicacao-loja/032.png)

Tem uma quantidade consideravel de opções sobre "Recursos financeiros", mas também não oferecemos nada relacionados a esse tipo de recurso.

![033.png](data/publicacao-loja/033.png)

Mesma resposta relacionado ao item "Apps de saúde".

![034.png](data/publicacao-loja/034.png)

#### Gerenciar a organização e a apresentação do app

Em "Configurações da loja", colocaremos tags para melhor organização do app na loja e adicionaremos dados para que os 
usuários possam entrar em contato.

![035.png](data/publicacao-loja/035.png)

Agora vem uma coisa que eu pessoalmente acho interessante. "Criar página 'Detalhes do app' padrão".

Nela colocaremos o nome do app, descrição e imagens.

![036.png](data/publicacao-loja/036.png)

![037.png](data/publicacao-loja/037.png)

![038.png](data/publicacao-loja/038.png)

#### Teste fechado

Como mencionado em algum ponto acima, o Google exige que um app publicado por pessoa fisica, passe por um teste fechado, então vamos fazer esta configuração.

![039.png](data/publicacao-loja/039.png)

Pensa numa parte complicada... Conseguir tantas pessoas dispostas a testar de verdade, não é nada fácil.

Agora vamos criar a versão de teste fechado.

Primeiro, precisamos enviar o arquivo gerado no momento do build, depois colocar o nome da versão e as notas de versão.

![040.png](data/publicacao-loja/040.png)

Após o preenchimento, apareceu um alerta de "problema", onde foi informado sobre mudanças na API do Android 13, mas como 
não tenho interesse em anúncios, a resposta simplesmente é "Não".

![041.png](data/publicacao-loja/041.png)

Agora basta enviar a "publicação" para revisão, e em breve os testadores poderão começar a testar

![042.png](data/publicacao-loja/042.png)

![043.png](data/publicacao-loja/043.png)

E 1 semana depois, está assim o feedback do Google.

![044.png](data/publicacao-loja/044.png)

Finalmente, após os 14 dias de teste, eis o feedback apresentado na plataforma.

![045.png](data/publicacao-loja/045.png)

Então, sem mais delongas (e sem nenhum e-mail informando que o período de teste foi concluído com sucesso), clicamos na opção "Solicitar a produção".

Assim que clicamos no botão, um questionário foi apresentado, relacionado ao teste fechado e feedback coletado com os testers.

![046.png](data/publicacao-loja/046.png)

![047.png](data/publicacao-loja/047.png)

![048.png](data/publicacao-loja/048.png)

Questionário respondido.

Agora temos mais um período de espera, em que o teste será feito pelo próprio google, e a informação sobre conclusão do teste chegará por e-mail e o prazo é de até 7 dias. 

![049.png](data/publicacao-loja/049.png)
<br /><small>Solicitação feita em 23/03 por volta das 19hs</small>

E depois de 2 dias, recebi e-mail de confirmação da aprovação do app.

![050.png](data/publicacao-loja/050.png)
<br /><small>E-mail de confirmação em 25/03 por volta das 15hs</small>

Agora, ao acessar o Painel no Google Play Console, foram liberadas 2 opções.

![051.png](data/publicacao-loja/051.png)

Vamos iniciar pelo país e região. Como o foco inicialmente é Brasil, procurei o país e selecionei.

![052.png](data/publicacao-loja/052.png)

Em seguida, vamos FINALMENTE criar a versão de produção.

Ao invés de clicar na opção "Criar uma nova versão", como os testes foram bem sucedidos, vou usar a mesma versão.

![053.png](data/publicacao-loja/053.png)

Foi apresentado a tela sobre os detalhes da versão. Essas informações já estavam corretas, pois foram adicionadas recentemente na última versão de testes, então apenas avencei neste ponto.

![054.png](data/publicacao-loja/054.png)

Em seguida, só cliquei em salvar novamente.

![055.png](data/publicacao-loja/055.png)


Basta clicar MAIS UMA VEZ para enviar as "mudanças"...

![056.png](data/publicacao-loja/056.png)

E aguardar por até (mais) 7 dias.

![057.png](data/publicacao-loja/057.png)

E após alguns instantes (literalmente não demorou 5 minutos), a página já mostrava o app como ativo.

![058.png](data/publicacao-loja/058.png)

E 

![059.png](data/publicacao-loja/059.png)

Politica de privacidade https://docs.google.com/document/d/e/2PACX-1vTyrNTW5XgrLViJAInDkH5uPLyQTFCs3T28kslJWvdEQnYM17mwkeno6caukoAzaWbA0D6Lf579fdyJ/pub
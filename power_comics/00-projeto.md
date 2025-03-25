# Power Comics, o projeto

## Idéia
Como um fã de longa data dos heróis coloridos inspirados (ou copiados se preferir chamar assim) dos orientais [Super Sentai](https://pt.wikipedia.org/wiki/Super_Sentai), com planos de acompanhar as revistas lançadas oficialmente pela [BOOM Studios](https://www.boom-studios.com), e agora sendo trazidas para o país por empresas como [IndieVisivel Press](https://indievisivelpress.com.br/?s=power+rangers) e [Pipoca & Nanquim](https://pipocaenanquim.com.br/colec-o/tartarugas-ninja.html), encontrei certa dificuldade em ter um checklist com ordens de leitura das HQs. Mesmo tendo o guia de quadrinhos do [MegaPower Brasil](https://www.megapowerbrasil.com/2017/05/o-guia-definitivo-dos-quadrinhos-de.html) (Grande abraço a todos do MegaPower, inclusive, por serem uma ótima referência e inspiração para este projeto), não senti que minhas necessidades foram atendidas como fã e futuro leitor dos quadrinhos com um guia prático para minha própria organização de leitura.

Levando em consideração essa MINHA necessidade de organização e de um checklist prático, decidi, como fã, criar algo que atenda à MINHA necessidade e, talvez, à de outros fãs da franquia, aproveitando a ideia também como fonte de estudos.

E foi assim que, fazendo papel do Alpha, recrutei um grupo de jovens que, na verdade, são recursos tecnológicos, dos quais cada um possui sua particularidade e apoio ao time de alguma forma.

### Rangers
   - Verde: [PHP](https://www.php.net). Esta linguagem de programação foi escolhida pela simplicidade de sua sintaxe e alto desempenho para aplicações WEB.
   - Vermelho: [Symfony](https://symfony.com). Como desenvolver algo em PHP puro estava fora de cogitação, pois seria reinventar a roda em diversos pontos e, também, por definir um prazo curto para o desenvolvimento, foi necessário pensar em um bom framework, que atendesse às necessidades do projeto e que fosse algo no qual eu me sinto carente de conhecimento, por ter muita coisa que não tive experiência no desenvolvimento. Acreditei ser uma ótima oportunidade de estudos.
   - Rosa: [API Platform](https://api-platform.com). Se tem uma coisa linda de se ver nesse projeto, é a linda aranha descendo ao consultar o endpoint /api/docs rsrsrsrsr. Agora sério. A decisão de usar este framework é principalmente pela sua robustez! Descrita como 'a plataforma API mais avançada, em qualquer framework ou idioma' pelo próprio criador do Symfony, ela será essencial para o desenvolvimento de nossa API REST, sendo o par perfeito para a segurança do sistema (autenticação JWT).
   - Amarela: [DART](https://dart.dev). Para levar nosso projeto para as mãos dos fãs, nada melhor do que usar a linguagem DART (desenvolvida pela Google) e usar sua maravilhosa compilação AOT (Compila o código para código nativo antes da execução, resultando em melhor desempenho) e JIT (Compila o código durante a execução, permitindo desenvolvimento rápido e eficiente).
   - Preto: [Flutter](https://flutter.dev). Este framework (também desenvolvido pela Google) permite criar interfaces nativas de alta performance para iOS, Android, WEB e Desktop. Foi o escolhido para desenvolver nosso futuro app Android.
   - Azul: [Docker](https://www.docker.com). Ainda não sabemos como ou onde será hospedado nosso projeto, mas para que todo o ambiente de desenvolvimento seja padronizado entre os possíveis desenvolvedores da API, usaremos Docker para dar todo o suporte necessário ao nosso projeto.

## Projetos
- [Power Comics API](./01-api.md)
- [Power Comics Mobile](./02-mobile.md)
- [Registro de dominio](./03-dominio.md)
- [Hospedagem da API](./04-hospedagem.md)
- [Conta Google Developer](./05-conta-google-developer.md)
- [Publicação na loja do google](./06-publicacao-loja-google.md)
- [Links de referências](./99-referencias.md)
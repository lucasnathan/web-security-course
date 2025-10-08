---
layout: default
title: Dia 2 - Autenticação e Design Inseguro
nav_order: 3
---

# Dia 2: Autenticação, Falhas Criptográficas e Design Inseguro

## Hoje vamos focar em exercícios práticos sobre autenticação, falhas criptográficas e design inseguro.

Mas primeiro vamos revisar alguns conceitos importantes.

## Requisições HTTP e HTTPS

HTTP (Hypertext Transfer Protocol) é o protocolo usado para comunicação entre clientes (como navegadores) e servidores web. Ele é baseado em texto e não criptografado, o que significa que os dados transmitidos podem ser interceptados e lidos por terceiros.

HTTPS (Hypertext Transfer Protocol Secure) é uma versão criptografada do protocolo HTTP. Ele utiliza SSL/TLS (Secure Sockets Layer/Transport Layer Security) para criptografar os dados transmitidos entre o cliente e o servidor, garantindo *confidencialidade e integridade*. Essas garantias são apenas para a origem, o destino e o canal (o que isso quer dizer?).

![WAT]({{ '/assets/images/wat.png' | relative_url }})

Isso significa que mesmo que os dados estejam protegidos, o protocolo não garante que você está se comunicando com a entidade correta (origem e destino). Além disso, o canal de comunicação pode ser comprometido, permitindo ataques como *man-in-the-middle*.

![MITM]({{ '/assets/images/mitm.png' | relative_url }})

## API (Application Programming Interface)

APIs são conjuntos de definições e protocolos que permitem que diferentes softwares se comuniquem entre si. Elas definem como os componentes de software devem interagir, facilitando a integração e a troca de dados entre sistemas. APIs restful são um estilo arquitetural para projetar APIs que utilizam os princípios do REST (Representational State Transfer). Elas são baseadas em recursos, que são representações de dados, e utilizam métodos HTTP (GET, POST, PUT, DELETE) para realizar operações sobre esses recursos.

![API]({{ '/assets/images/api.png' | relative_url }})

## CRUD 

![CRUD]({{ '/assets/images/crud.png' | relative_url }})

CRUD é um acrônimo que representa as quatro operações básicas que podem ser realizadas em um banco de dados ou sistema de armazenamento de dados:
- **Create (Criar)**: Adicionar novos dados ou registros ao sistema.
- **Read (Ler)**: Recuperar ou visualizar dados existentes no sistema.
- **Update (Atualizar)**: Modificar ou alterar dados existentes no sistema.
- **Delete (Excluir)**: Remover dados ou registros do sistema.

## Exercício de falha de Autenticação

Neste exercício, você irá explorar uma vulnerabilidade de autenticação usando a ferramenta Burp Suite. A vulnerabilidade está relacionada ao cabeçalho `Host` na requisição HTTP, que pode ser manipulado para obter acesso não autorizado a áreas restritas do site.

link do lab: [https://portswigger.net/web-security/host-header/exploiting/lab-host-header-authentication-bypass](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-authentication-bypass)

Lab: Host header authentication bypass

### Resolução do Laboratório

1.  Envie a requisição `GET /` que recebeu uma resposta 200 para o Burp Repeater. Observe que você pode alterar o cabeçalho `Host` para um valor arbitrário e ainda assim acessar a página inicial com sucesso.
2.  Acesse `/robots.txt` e observe que há um painel de administração em `/admin`.
3.  Tente acessar `/admin`. Você não terá acesso, mas observe a mensagem de erro, que revela que o painel pode ser acessado por usuários locais.
4.  Envie a requisição `GET /admin` para o Burp Repeater.
5.  No Burp Repeater, altere o cabeçalho `Host` para `localhost` e envie a requisição. Observe que agora você acessou com sucesso o painel de administração, que oferece a opção de excluir diferentes usuários.
6.  Altere a linha da requisição para `GET /admin/delete?username=carlos` e envie a requisição para excluir o usuário `carlos` e resolver o laboratório.


## Exercício de Design Inseguro

Neste exercício, você irá explorar uma vulnerabilidade de design inseguro usando a ferramenta Burp Suite. A vulnerabilidade está relacionada à confiança excessiva nos controles do lado do cliente, permitindo que você manipule os dados enviados ao servidor para obter um benefício não autorizado.

[https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)  

Este laboratório não valida adequadamente a entrada do usuário. Você pode explorar uma falha lógica em seu fluxo de compra para comprar itens por um preço não intencional. Para resolver o laboratório, compre uma "Lightweight l33t leather jacket".

Você pode fazer login em sua própria conta usando as seguintes credenciais: `wiener:peter`

### Resolução do Laboratório

1.  Com o Burp em execução, faça login e tente comprar a jaqueta de couro. O pedido é rejeitado porque você não tem crédito suficiente na loja.
2.  No Burp, vá para "Proxy" - "HTTP history" e estude o processo de pedido. Observe que, ao adicionar um item ao seu carrinho, a requisição correspondente contém um parâmetro `price`. Envie a requisição `POST /cart` para o Burp Repeater.
3.  No Burp Repeater, altere o `price` para um valor inteiro arbitrário e envie a requisição. Atualize o carrinho e confirme que o preço foi alterado com base na sua entrada.
4.  Repita este processo para definir o preço para qualquer valor menor que o seu crédito disponível na loja.
5.  Conclua o pedido para resolver o laboratório.

### Exemplo:

[https://infosecwriteups.com/how-i-found-a-ticket-booking-bug-that-allowed-me-to-travel-almost-for-free-in-tnstc-2c7aa23aebf6](https://infosecwriteups.com/how-i-found-a-ticket-booking-bug-that-allowed-me-to-travel-almost-for-free-in-tnstc-2c7aa23aebf6)



## Exercício de Criptografia Fraca

Neste exercício, você irá explorar uma vulnerabilidade de criptografia fraca usando a ferramenta Burp Suite. A vulnerabilidade está relacionada ao uso inseguro de um algoritmo de criptografia para proteger cookies, permitindo que você manipule os cookies para obter acesso não autorizado.

[https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle)

Este laboratório contém uma falha lógica que expõe um oráculo de criptografia aos usuários. Para resolver o laboratório, explore essa falha para obter acesso ao painel de administração e excluir o usuário **carlos**.

Você pode fazer login em sua própria conta usando as seguintes credenciais: `wiener:peter`

### Resolução do Laboratório

1.  Faça login com a opção "Permanecer conectado" ativada e poste um comentário. Estude as requisições e respostas correspondentes usando as ferramentas manuais do Burp. Observe que o cookie `stay-logged-in` está criptografado.
2.  Note que, ao tentar enviar um comentário com um endereço de e-mail inválido, a resposta define um cookie de notificação (`notification`) criptografado antes de redirecioná-lo para o post do blog.
3.  Observe que a mensagem de erro reflete sua entrada do parâmetro `email` em texto claro: `Invalid email address: seu-email-invalido`.
4.  Deduz-se que isso deve ser descriptografado a partir do cookie de notificação. Envie a requisição `POST /post/comment` e a subsequente `GET /post?postId=x` (contendo o cookie de notificação) para o Burp Repeater.
5.  No Repeater, observe que você pode usar o parâmetro `email` da requisição `POST` para criptografar dados arbitrários e refletir o texto cifrado correspondente no cabeçalho `Set-Cookie`. Da mesma forma, você pode usar o cookie `notification` na requisição `GET` para descriptografar um texto cifrado arbitrário e refletir a saída na mensagem de erro. Para simplificar, clique duas vezes na aba de cada requisição e renomeie-as para `encrypt` e `decrypt`, respectivamente.
6.  Na requisição `decrypt`, copie seu cookie `stay-logged-in` e cole-o no cookie `notification`. Envie a requisição. Em vez da mensagem de erro, a resposta agora contém o cookie `stay-logged-in` descriptografado, por exemplo: `wiener:1598530205184`.
7.  Isso revela que o cookie deve estar no formato `username:timestamp`. Copie o timestamp para a sua área de transferência.
8.  Vá para a requisição `encrypt` e altere o parâmetro `email` para `administrator:seu-timestamp`. Envie a requisição e copie o novo cookie de notificação da resposta.
9.  Descriptografe este novo cookie e observe que o prefixo de 23 caracteres "Invalid email address: " é adicionado automaticamente a qualquer valor que você passa usando o parâmetro `email`. Envie o cookie de notificação para o Burp Decoder.
10. No Decoder, faça o URL-decode e o Base64-decode do cookie.
11. No Burp Repeater, mude para a aba "Hex" do editor de mensagens. Selecione os primeiros 23 bytes, clique com o botão direito e selecione "Delete selected bytes".
12. Recodifique os dados e copie o resultado para o cookie `notification` da requisição `decrypt`. Ao enviar a requisição, observe que uma mensagem de erro indica que um algoritmo de criptografia baseado em blocos é usado e que o comprimento da entrada deve ser um múltiplo de 16. Você precisa preencher o prefixo "Invalid email address: " com bytes suficientes para que o número de bytes que você removerá seja um múltiplo de 16.
13. No Burp Repeater, volte para a requisição `encrypt` e adicione 9 caracteres ao início do valor do cookie desejado, por exemplo: `xxxxxxxxxadministrator:seu-timestamp`.
14. Criptografe esta entrada e use a requisição `decrypt` para testar se ela pode ser descriptografada com sucesso.
15. Envie o novo texto cifrado para o Decoder, depois faça o URL-decode e o Base64-decode. Desta vez, exclua 32 bytes do início dos dados. Recodifique os dados e cole-os no parâmetro `notification` na requisição `decrypt`. Verifique a resposta para confirmar que sua entrada foi descriptografada com sucesso e, crucialmente, não contém mais o prefixo "Invalid email address: ". Você deve ver apenas `administrator:seu-timestamp`.
16. Do histórico do Proxy, envie a requisição `GET /` para o Burp Repeater. Exclua completamente o cookie de sessão e substitua o cookie `stay-logged-in` pelo texto cifrado do seu cookie criado. Envie a requisição. Observe que agora você está logado como administrador e tem acesso ao painel de administração.
17. Usando o Burp Repeater, acesse `/admin` e observe a opção para excluir usuários. Acesse `/admin/delete?username=carlos` para resolver o laboratório.

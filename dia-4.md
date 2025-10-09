---
layout: default
title: Dia 4 - Ataques Avançados
nav_order: 5
---

# Dia 4: Ataques Avançados - CSRF, SSRF, XXE e OAuth

Hoje vamos avançar nos conceitos que já aprendemos nos exercícios passados. O foco das vulnerabilidades que vamos conversar hoje é enganar o servidor, forçar ações em nome do usuário e abusar da confiança entre sistemas.

Vamos discutir sobre 3 tópicos:
- **Cross-Site Request Forgery (CSRF):** O ataque que faz o usuário trabalhar para o hacker sem saber.
- **Server-Side Request Forgery (SSRF):** Quando você convence o servidor a fazer o trabalho sujo por você.
- **Falhas em OAuth 2.0:** Como o "Login com Google" pode abrir portas para o mal.

## Cross-Site Request Forgery (CSRF)

Imagine que você está logado no seu internet banking. Seu navegador guarda um "crachá" (o cookie de sessão) que te identifica para o banco. Agora, você recebe um e-mail com um link para uma foto de gatinho fofo. Ao clicar, você não vê só o gatinho. Escondido na página, há um código que envia uma requisição para o seu banco, mandando transferir R$ 1.000 para a conta de um atacante.

Como você estava logado, seu navegador, muito prestativo, anexa seu "crachá" àquela requisição maliciosa. O banco vê o crachá, confirma que é você e... autoriza a transferência. Você acabou de ser vítima de um ataque de **CSRF (Falsificação de Solicitação entre Sites)**.

O CSRF explora a confiança que a aplicação tem no seu navegador. Ele não rouba seu crachá, ele o usa contra você!

A maneira mais simples de realizar esta ação é através de um formulário HTML que é automaticamente submetido quando a página é carregada:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSRF Attack</title>
</head>
<body onload="document.forms[0].submit()">
    <form action="https://seu-banco.com/transfer" method="POST">
        <input type="hidden" name="amount" value="1000">
        <input type="hidden" name="to_account" value="conta-do-atacante">
    </form>
</body>
</html>
```

Ou utilizando um iframe invisível:

```html
<iframe src="https://seu-banco.com/transfer?amount=1000&to_account=conta-do-atacante" style="display:none;"></iframe>
```

### Riscos

Os riscos do CSRF são altos, especialmente em aplicações que realizam ações sensíveis (transferências, mudanças de senha, etc.) sem medidas de proteção adequadas. Um ataque bem-sucedido pode resultar em:
- Transferência de dinheiro
- Mudança de e-mail ou senha
- Exclusão de contas ou dados importantes
- Realização de compras ou outras ações em nome do usuário

---


### Prevenção

A principal defesa é o **Token Anti-CSRF**. Funciona assim:
1.  Quando você acessa a página de transferência, o banco te dá, além do formulário, um "código secreto" único e imprevisível (o token), que é passado como um campo na requisição. No backend, esse token é normalmente gerado utilizando uma função de hash criptográfica, garantindo que seja único e difícil de adivinhar, além de ser ligado à sessão do usuário.
2.  Ao enviar o formulário, seu navegador envia o pedido de transferência junto com esse código secreto.
3.  O servidor do banco confere se o código secreto recebido é o mesmo que ele te deu.

O site do gatinho fofo não tem como saber qual é o seu código secreto, então a requisição forjada falhará. Outras medidas incluem o uso do atributo `SameSite` nos cookies, que instrui o navegador a não enviar o "crachá" para requisições vindas de outros sites.


---

## Server-Side Request Forgery (SSRF)

Agora, imagine uma aplicação que permite importar uma foto a partir de uma URL. Você cola o link da imagem e o servidor da aplicação vai até lá, baixa a imagem e a exibe para você.

Um atacante pensa: "E se, em vez de uma URL de imagem, eu colocar o endereço de um sistema interno da empresa, como `http://localhost/admin` ou `http://192.168.1.2/database-backup`?".

Se a aplicação não validar a URL corretamente, o servidor, que está dentro da rede protegida da empresa, fará essa requisição. Isso é o **SSRF (Falsificação de Solicitação do Lado do Servidor)**. O atacante usa o servidor da aplicação como um "laranja" para atacar sistemas que ele não conseguiria alcançar diretamente da internet. É um proxy involuntário!

A partir daí, o atacante pode explorar essa falha para acessar serviços internos, roubar dados sensíveis ou até comprometer outros sistemas na rede.

### Riscos

O estrago pode ser enorme:
- **Mapeamento da Rede Interna:** O atacante pode usar o servidor para escanear portas e descobrir outros sistemas na rede interna.
- **Acesso a Dados Sensíveis:** É possível acessar painéis de administração, bancos de dados ou serviços internos que não possuem autenticação por confiarem que só seriam acessados de dentro da rede.
- **Roubo de Credenciais na Nuvem:** Em provedores como AWS, GCP e Azure, existe um serviço de metadados em um endereço local. Um SSRF pode ser usado para acessar esse serviço e roubar credenciais temporárias da máquina, dando ao atacante controle sobre a infraestrutura na nuvem.

---

## Falhas de Implementação em OAuth 2.0

OAuth 2.0 é um padrão de autorização, não de autenticação. O que significa que ele é usado para permitir que uma aplicação (cliente) acesse recursos em nome do usuário. Ele nasceu em 2006 para resolver o problema de compartilhar credenciais entre serviços, permitindo que um usuário autorize uma aplicação a agir em seu nome sem entregar sua senha. Antes dele, tinha a versão 1.0, que era mais complexa e difícil de implementar, entretanto essa versão ainda é usada em alguns serviços. A **RFC 6749** é a especificação oficial do OAuth 2.0. Aliás, rfc significa **"Request for Comments"** (Pedido de Comentários) e é um tipo de documento usado para descrever padrões, protocolos e tecnologias na internet ele é mantido pela IETF (Internet Engineering Task Force) que é uma organização aberta que desenvolve e promove padrões voluntários da internet formada por engenheiros, pesquisadores, desenvolvedores e outros profissionais da área de tecnologia.

O OAuth 2.0 é usado frequentemente para implementar logins sociais ("Login com..."). Falhas na sua implementação podem permitir que um atacante se passe por uma vítima e acesse seus dados na aplicação cliente.

![Fluxo OAuth 2.0]({{ '/assets/images/oauth20.png' | relative_url }})

### Falhas Comuns

- **Validação inadequada do `redirect_uri`**: Permite que o `authorization_code` seja enviado para um servidor controlado pelo atacante.
- **Falta do parâmetro `state`**: Deixa a aplicação vulnerável a ataques de CSRF durante o processo de login, permitindo que um atacante associe sua conta do serviço de identidade (ex: Google) à conta da vítima na aplicação cliente.

### Riscos

Os riscos incluem:
- **Sequestro de Conta**: Um atacante pode obter acesso à conta da vítima na aplicação cliente.
- **Exposição de Dados Sensíveis**: Dados pessoais e informações confidenciais podem ser acessados.
- **Acesso Não Autorizado**: O atacante pode realizar ações em nome da vítima.



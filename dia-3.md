---
layout: default
title: Dia 3 - Injeção, Falhas de Configuração e XSS
nav_order: 4
---

# Dia 3: Injeção, Falhas de Configuração e XSS

## Hoje vamos focar em exercícios práticos sobre Injeção de SQL, Falhas de Configuração e XSS (Cross-Site Scripting).

Vamos revisar rapidamente os conceitos antes de partir para os laboratórios.

## Injeção de SQL (SQLi)

A injeção de SQL é uma vulnerabilidade que permite a um atacante interferir nas consultas que uma aplicação faz ao seu banco de dados. Geralmente, permite que um atacante visualize dados que não deveria ser capaz de ver, incluindo dados de outros usuários ou quaisquer outros dados que a própria aplicação possa acessar. Em muitos casos, um atacante pode modificar ou excluir esses dados, causando alterações persistentes no conteúdo ou comportamento da aplicação.

## Detectando SQLi

Voce pode detectar vulnerabilidades de SQLi tentando inserir caracteres especiais em campos de entrada, como aspas simples (`'`), aspas duplas (`"`), ou comentários SQL (`--`). Se a aplicação retornar um erro de banco de dados ou um comportamento inesperado, isso pode indicar uma vulnerabilidade. As forma mais conhecidas são:
- **Injeção de SQL baseada em erros:** O atacante insere uma carga útil que causa um erro de banco de dados, revelando informações sobre a estrutura do banco de dados. Para isso inserir uma aspa simples (`'`) em um campo de entrada pode ser suficiente, na prática você esta tentando quebrar a consulta SQL, dizendo ao banco de dados que a string não foi fechada corretamente.
- **Injeção de SQL cega (Blind SQL Injection):** O atacante faz perguntas ao banco de dados e observa as respostas (verdadeiro/falso) para inferir informações como: (`OR 1=1`), sem verer diretamente os dados.
- **Injeção de SQL baseada em tempo:** O atacante insere uma carga útil que faz o banco de dados atrasar sua resposta, permitindo inferir informações com base no tempo de resposta.

Para explorar vulnerabilidades de sqli você pode consultar este cheat sheet: [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

### Prevenção

Para previnir a injeção de SQL, é importante usar consultas parametrizadas (prepared statements), stored procedures, validação e sanitização de entradas, além de aplicar o princípio do menor privilégio no banco de dados.

![SQLi]({{ '/assets/images/sqli.png' | relative_url }})

## XSS (Cross-Site Scripting)

Cross-Site Scripting (XSS) é uma vulnerabilidade de segurança web que permite a um atacante injetar scripts maliciosos (geralmente JavaScript) em páginas web vistas por outros usuários. Esses scripts podem roubar informações sensíveis, como cookies de sessão, ou realizar ações em nome do usuário sem o seu consentimento.

Existem três tipos principais de XSS:
- **Refletido (Reflected XSS):** O script malicioso é refletido na resposta do servidor, geralmente através de parâmetros de URL ou formulários.
- **Armazenado (Stored XSS):** O script malicioso é armazenado no servidor (por exemplo, em um banco de dados) e é servido a todos os usuários que acessam a página vulnerável.
- **DOM-based XSS:** O script malicioso manipula o Document Object Model (DOM) da página diretamente no navegador, sem a necessidade de interação com o servidor.

### Detectando XSS

Você pode detectar vulnerabilidades de XSS inserindo scripts simples em campos de entrada, como `<script>alert('XSS')</script>`, e observando se o script é executado quando a página é carregada. Se o alerta aparecer, a aplicação é vulnerável a XSS. Teste também com variações como `<img src=x onerror=alert('XSS')>` para verificar diferentes contextos de injeção.

Para explorar vulnerabilidades de xss você pode consultar este cheat sheet: [https://portswigger.net/web-security/cross-site-scripting/cheat-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

### Prevenção

Para previnir a injeção de XSS, é importante validar e sanitizar todas as entradas do usuário, usar cabeçalhos de segurança como Content Security Policy (CSP) e escapar corretamente os dados antes de inseri-los em páginas HTML.

![XSS]({{ '/assets/images/xss.png' | relative_url }})


## Falhas de Configuração (Security Misconfiguration)

Falhas de configuração de segurança são uma das vulnerabilidades mais comuns. Elas podem ocorrer em qualquer nível da pilha de uma aplicação, incluindo a plataforma, o servidor web, o servidor de aplicação, o banco de dados, o framework e o código personalizado. Exemplos comuns incluem:

- Deixar contas padrão com senhas inalteradas.
- Expor arquivos ou diretórios sensíveis (como `.git` ou `.svn`).
- Mensagens de erro excessivamente detalhadas que revelam informações sobre a infraestrutura.
- Não aplicar patches de segurança ou manter os sistemas desatualizados.

### Prevenção

![Security Misconfiguration]({{ '/assets/images/rtfm.png' | relative_url }})

Para prevenir falhas de configuração, é importante seguir as melhores práticas de segurança, como desabilitar funcionalidades desnecessárias, aplicar patches regularmente, usar configurações seguras por padrão e revisar regularmente as configurações de segurança.


## Exercício de Injeção de SQL

Neste exercício, você explorará uma vulnerabilidade de injeção de SQL simples em uma cláusula `WHERE`, que permite contornar filtros e recuperar dados ocultos do banco de dados. A query utilizada no lab é algo como:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1;
```

Para resolver o lab, você precisará modificar o parâmetro `category` para injetar uma condição que seja sempre verdadeira, permitindo que todos os produtos sejam retornados, incluindo aqueles que não foram lançados (`released = 0`). Ou seja, mostrar produtos que não deveriam ser visíveis. Qual o tipo de SQLi que estamos explorando?

Link do lab: https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data


### Resolução do Laboratório

1.  Acesse o laboratório e observe que a aplicação exibe uma lista de produtos filtrados por categoria.
2.  Clique em uma das categorias, por exemplo, "Gifts", e observe que a URL muda para `.../filter?category=Gifts`. Isso indica que o parâmetro `category` está sendo usado para filtrar os resultados.
3.  Use o Burp Suite para interceptar a requisição `GET /filter?category=Gifts` e envie-a para o Burp Repeater.
4.  Na requisição no Repeater, modifique o valor do parâmetro `category` para injetar uma condição que seja sempre verdadeira. Uma carga útil comum para isso é `' OR 1=1 --`. A nova URL ficaria: `/filter?category=' OR 1=1 --`.
5.  Envie a requisição. A consulta SQL no backend provavelmente se parecerá com `SELECT * FROM products WHERE category = '' OR 1=1 --' AND released = 1`. O `OR 1=1` torna a condição sempre verdadeira, e o `--` comenta o resto da consulta, fazendo com que todos os produtos, incluindo os não lançados (`released = 0`), sejam retornados.
6.  Observe que a resposta agora inclui produtos que não estavam visíveis anteriormente. O laboratório será resolvido ao exibir esses produtos ocultos.

---

## Exercício de Falha de Configuração 

Neste exercício, você explorará uma falha de configuração que expõe o histórico de um sistema de controle de versão (Git), permitindo o acesso a informações sensíveis. Para resolver o lab voce deverá logar como administrator e deletar o usuario carlos.

Link do lab: [https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history)


### Resolução do Laboratório

1.  Acesse o laboratório e navegue pela aplicação.
2.  Suspeite que a aplicação pode ter um diretório `.git` exposto na raiz do servidor web. Tente acessar `/.git`.
3.  O servidor responderá com um erro 404 Not Found, mas isso não significa que o diretório não exista. Ferramentas de download recursivo podem ser capazes de baixar o conteúdo.
4.  Para confirmar a existência, tente acessar um arquivo comum dentro do diretório `.git`, como `/.git/config`. O servidor deve retornar o conteúdo deste arquivo.
5.  Com a confirmação, use uma ferramenta como `wget` para baixar o repositório. Exemplo: `wget -r http://<LAB-ID>.web-security-academy.net/.git/`.
6.  Navegue até o diretório onde o repositório foi baixado.
7.  Use o comando `git log` para visualizar o histórico de commits. Você verá um commit com a mensagem "Remove admin password from config".
8.  Use `git diff <COMMIT_ID>` para ver as alterações feitas nesse commit. Isso revelará a senha do administrador que foi removida.
9.  Faça login na aplicação com o usuário `administrator` e a senha encontrada para resolver o laboratório.

---

## Exercício de XSS (Cross-Site Scripting)

Neste exercício, você explorará uma vulnerabilidade de XSS, onde a aplicação recebe dados em uma requisição HTTP e os inclui na resposta imediata de forma insegura. Qual o tipo de XSS que estamos explorando?

Link do lab: [https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)


### Resolução do Laboratório

1.  Acesse o laboratório e observe que há uma funcionalidade de busca.
2.  Na caixa de busca, digite uma string de teste, como `12345`, e realize a pesquisa.
3.  Observe que a URL agora contém sua string de busca (`/?search=12345`) e a página exibe "0 search results for '12345'". Isso indica que a entrada do usuário está sendo refletida na página.
4.  Para testar a vulnerabilidade, injete uma tag HTML simples no parâmetro de busca, como `/?search=<h1>TESTE</h1>`. A página deve renderizar a palavra "TESTE" como um título de nível 1, confirmando que o HTML é processado.
5.  Agora, injete um script malicioso para criar um pop-up de alerta. A carga útil (payload) será `<script>alert(1)</script>`.
6.  A URL final ficará parecida com: `.../?search=<script>alert(1)</script>`.
7.  Acesse essa URL. Se um pop-up de alerta com o número "1" aparecer na sua tela, significa que você explorou com sucesso a vulnerabilidade de XSS refletido, e o laboratório será resolvido.

### Riscos

Se um atacante controla um script que é executado no navegador de outro usuário, ele pode roubar cookies de sessão, capturar entradas do teclado, redirecionar o usuário para sites maliciosos, ou realizar ações em nome do usuário sem o seu consentimento.
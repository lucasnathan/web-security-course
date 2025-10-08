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

![SQLi]({{ '/assets/images/sqli.png' | relative_url }})

## Falhas de Configuração (Security Misconfiguration)

Falhas de configuração de segurança são uma das vulnerabilidades mais comuns. Elas podem ocorrer em qualquer nível da pilha de uma aplicação, incluindo a plataforma, o servidor web, o servidor de aplicação, o banco de dados, o framework e o código personalizado. Exemplos comuns incluem:

- Deixar contas padrão com senhas inalteradas.
- Expor arquivos ou diretórios sensíveis (como `.git` ou `.svn`).
- Mensagens de erro excessivamente detalhadas que revelam informações sobre a infraestrutura.
- Não aplicar patches de segurança ou manter os sistemas desatualizados.

## XSS (Cross-Site Scripting)

Cross-Site Scripting (XSS) é uma vulnerabilidade de segurança web que permite a um atacante injetar scripts maliciosos (geralmente JavaScript) em páginas web vistas por outros usuários. Esses scripts podem roubar informações sensíveis, como cookies de sessão, ou realizar ações em nome do usuário sem o seu consentimento.

![XSS]({{ '/assets/images/xss.png' | relative_url }})

---

## Exercício de Injeção de SQL

Neste exercício, você explorará uma vulnerabilidade de injeção de SQL simples em uma cláusula `WHERE`, que permite contornar filtros e recuperar dados ocultos do banco de dados.

Link do lab: https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

### Resolução do Laboratório

1.  Acesse o laboratório e observe que a aplicação exibe uma lista de produtos filtrados por categoria.
2.  Clique em uma das categorias, por exemplo, "Gifts", e observe que a URL muda para `.../filter?category=Gifts`. Isso indica que o parâmetro `category` está sendo usado para filtrar os resultados.
3.  Use o Burp Suite para interceptar a requisição `GET /filter?category=Gifts` e envie-a para o Burp Repeater.
4.  Na requisição no Repeater, modifique o valor do parâmetro `category` para injetar uma condição que seja sempre verdadeira. Uma carga útil comum para isso é `' OR 1=1 --`. A nova URL ficaria: `/filter?category=' OR 1=1 --`.
5.  Envie a requisição. A consulta SQL no backend provavelmente se parecerá com `SELECT * FROM products WHERE category = '' OR 1=1 --' AND released = 1`. O `OR 1=1` torna a condição sempre verdadeira, e o `--` comenta o resto da consulta, fazendo com que todos os produtos, incluindo os não lançados (`released = 0`), sejam retornados.
6.  Observe que a resposta agora inclui produtos que não estavam visíveis anteriormente. O laboratório será resolvido ao exibir esses produtos ocultos.

---

## Exercício de Falha de Configuração

Neste exercício, você explorará uma falha de configuração que expõe o histórico de um sistema de controle de versão (Git), permitindo o acesso a informações sensíveis.

Link do lab: https://portswigger.net/web-security/security-misconfiguration/lab-infoleak-in-version-control-history

Lab: Information disclosure in version control history

### Resolução do Laboratório

1.  Acesse o laboratório e navegue pela aplicação.
2.  Suspeite que a aplicação pode ter um diretório `.git` exposto na raiz do servidor web. Tente acessar `/.git`.
3.  O servidor responderá com um erro 404 Not Found, mas isso não significa que o diretório não exista. Ferramentas de download recursivo podem ser capazes de baixar o conteúdo.
4.  Para confirmar a existência, tente acessar um arquivo comum dentro do diretório `.git`, como `/.git/config`. O servidor deve retornar o conteúdo deste arquivo.
5.  Com a confirmação, use uma ferramenta como `git-dumper` ou `GitTools` para baixar o repositório. Exemplo: `git-dumper http://<LAB-ID>.web-security-academy.net/ .git/ ~/git-dump`.
6.  Navegue até o diretório onde o repositório foi baixado (`~/git-dump`).
7.  Use o comando `git log` para visualizar o histórico de commits. Você verá um commit com a mensagem "Remove admin password from config".
8.  Use `git diff <COMMIT_ID>` para ver as alterações feitas nesse commit. Isso revelará a senha do administrador que foi removida.
9.  Faça login na aplicação com o usuário `administrator` e a senha encontrada para resolver o laboratório.

---

## Exercício de XSS (Cross-Site Scripting)

Neste exercício, você explorará uma vulnerabilidade de XSS refletido simples, onde a aplicação recebe dados em uma requisição HTTP e os inclui na resposta imediata de forma insegura.

Link do lab: https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded

Lab: Reflected XSS into HTML context with nothing encoded

### Resolução do Laboratório

1.  Acesse o laboratório e observe que há uma funcionalidade de busca.
2.  Na caixa de busca, digite uma string de teste, como `12345`, e realize a pesquisa.
3.  Observe que a URL agora contém sua string de busca (`/?search=12345`) e a página exibe "0 search results for '12345'". Isso indica que a entrada do usuário está sendo refletida na página.
4.  Para testar a vulnerabilidade, injete uma tag HTML simples no parâmetro de busca, como `/?search=<h1>TESTE</h1>`. A página deve renderizar a palavra "TESTE" como um título de nível 1, confirmando que o HTML é processado.
5.  Agora, injete um script malicioso para criar um pop-up de alerta. A carga útil (payload) será `<script>alert(1)</script>`.
6.  A URL final ficará parecida com: `.../?search=<script>alert(1)</script>`.
7.  Acesse essa URL. Se um pop-up de alerta com o número "1" aparecer na sua tela, significa que você explorou com sucesso a vulnerabilidade de XSS refletido, e o laboratório será resolvido.
---
layout: post
title: "Relatório de Pentest - Information Disclosure in Error Messages"
author: "Nathan"
date: 2025-02-10
categories: relatorio pentest
---

# Relatório de Teste de Intrusão: Information Disclosure in Error Messages

## 1. Resumo Executivo

*Este relatório detalha os resultados de um teste de intrusão realizado na aplicação Web Security Academy. O teste identificou uma vulnerabilidade de divulgação de informações que expõe a versão de um framework utilizado no backend, representando um risco para a segurança da aplicação.*

**Escopo:** A aplicação web hospedada em `https://0a40002a04f1cb6f83da978a0011002f.web-security-academy.net/`.
**Período do Teste:** 10/02/2025.
**Principal Descoberta:** Divulgação de informações sensíveis (versão de framework) em mensagens de erro.
**Nível de Risco Geral:** Médio.
**Conclusão:** A aplicação não trata adequadamente os erros, revelando detalhes técnicos internos que podem ser explorados por atacantes para pesquisar vulnerabilidades conhecidas e direcionar ataques mais sofisticados. Recomenda-se a implementação de páginas de erro genéricas.

---

## 2. Metodologia Utilizada

- **Ferramentas:** Burp Suite Community, etc.
- **Etapas Seguidas:**
    1.  **Reconhecimento:** Análise inicial da aplicação para entender suas funcionalidades, como a visualização de produtos.
    2.  **Mapeamento:** Identificação de endpoints e parâmetros, como o endpoint `/product` e o parâmetro `productId`.
    3.  **Análise e Exploração de Vulnerabilidades:** Envio de requisições com dados malformados para o parâmetro `productId` com o objetivo de induzir a aplicação a um estado de erro.
    4.  **Pós-exploração (se aplicável):** Não foi aplicável para esta vulnerabilidade.

---

## 3. Vulnerabilidades Identificadas

*Esta é a seção principal do relatório. Para cada vulnerabilidade encontrada, crie uma subseção detalhando as informações abaixo.*

### 3.1. Divulgação de Informações em Mensagens de Erro

- **Descrição:** A aplicação exibe mensagens de erro detalhadas que revelam a versão de um framework utilizado no servidor. Ao enviar um valor inesperado para o parâmetro `productId`, a resposta do servidor inclui uma stack trace que expõe a tecnologia e a versão específica. Essa informação permite que um atacante pesquise por vulnerabilidades conhecidas (CVEs) para essa versão, facilitando a exploração de outras falhas.
- **Localização:** Parâmetro `productId` no endpoint `/product`.
- **Passos para Reprodução:**
    1.  Navegar até a página de um produto qualquer.
    2.  Interceptar a requisição `GET /product?productId=...` utilizando o Burp Suite.
    3.  Modificar o valor do parâmetro `productId` para uma string malformada, como `1rq69t<script>alert(1)</script>qiher`.
    4.  Enviar a requisição modificada.
    5.  Analisar a resposta do servidor e observar a mensagem de erro detalhada, que contém a versão do framework.
- **Evidências (Prova de Conceito):** A resposta HTTP a uma requisição malformada continha uma mensagem de erro indicando a versão do framework, como "Apache Struts 2.3.31".
    ```http
    HTTP/2 500 Internal Server Error
    Content-Type: text/html;charset=utf-8
    Content-Language: en
    Content-Length: 4345
    ...
    
    Apache Struts 2.5.12 - ... Exception ...
    ```
- **Análise de Risco (CVSS 3.1):**
    - **Vetor de Ataque (AV):** Network (N)
    - **Complexidade do Ataque (AC):** Low (L)
    - **Privilégios Necessários (PR):** None (N)
    - **Interação do Usuário (UI):** None (N)
    - **Escopo (S):** Unchanged (U)
    - **Impacto (Confidencialidade/Integridade/Disponibilidade):** Low (L) / None (N) / None (N)
    - **Pontuação Base:** 5.3
    - **Nível de Severidade:** Médio
- **Recomendações para Mitigação:** Configurar o servidor web e a aplicação para não exibir mensagens de erro detalhadas ou stack traces para o usuário final. Em vez disso, devem ser apresentadas páginas de erro genéricas e os detalhes do erro devem ser registrados em logs internos para análise pela equipe de desenvolvimento.

*(Copie e cole esta estrutura para cada vulnerabilidade adicional encontrada)*

---

## 4. Conclusão

O teste de intrusão revelou uma falha de segurança de risco médio relacionada à divulgação de informações. Embora essa vulnerabilidade por si só não comprometa diretamente os dados, ela fornece informações valiosas que podem ser o primeiro passo para um ataque bem-sucedido. É crucial que a recomendação de implementar o tratamento adequado de erros seja aplicada para fortalecer a postura de segurança da aplicação e reduzir a superfície de ataque.
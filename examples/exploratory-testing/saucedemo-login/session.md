# Relatório de Sessão — Login do Sauce Demo

| Início | Testador | Módulo / alvo | Ambiente | Duração |
|---|---|---|---|---|
| 2026-10-26 (horário não registrado) | GitHub Copilot (agente de IA) | Login — Sauce Demo | https://www.saucedemo.com/ · Playwright MCP; navegador e dispositivo não registrados | Não registrada; 6 experimentos |

## Charter
*Explore* a interface de login do Sauce Demo  
*Com* campos vazios, preenchimento parcial, espaços em branco e credenciais sintéticas inválidas  
*Para descobrir* como a interface valida campos obrigatórios, apresenta mensagens de erro e responde a entradas inválidas, observando campos, URL e redirecionamentos sem usar credenciais válidas.

**Escopo:** somente a interface de login. Foram realizados seis envios controlados. Não foram fornecidos requisitos ou critérios de aceitação para comparar os comportamentos.

## Experimentos executados

| ID | Entrada | Resultado observado |
|---|---|---|
| 1 | Usuário vazio; senha vazia | `Epic sadface: Username is required` |
| 2 | Usuário `qa-explore-invalid-user-01`; senha vazia | `Epic sadface: Password is required`; o usuário permaneceu no campo. |
| 3 | Usuário vazio; senha `qa-explore-invalid-password-01` | `Epic sadface: Username is required`; a senha permaneceu preenchida e mascarada. |
| 4 | Usuário e senha contendo somente espaços (`   `) | `Epic sadface: Username and password do not match any user in this service` |
| 5 | Usuário ` qa-explore-invalid-user-02 `; senha ` qa-explore-invalid-password-02 ` | `Epic sadface: Username and password do not match any user in this service`; os valores com espaços nas extremidades permaneceram nos campos. |
| 6 | Usuário `qa-explore-invalid-user-03`; senha `qa-explore-invalid-password-03` | `Epic sadface: Username and password do not match any user in this service`; os valores permaneceram nos campos. |

## Notas
*(I) Informação · (R) Risco*

- (I) Com ambos os campos vazios, a interface exibiu `Epic sadface: Username is required`.
- (I) Com somente um campo preenchido, a mensagem identificou o campo ausente: `Epic sadface: Password is required` quando só o usuário estava preenchido e `Epic sadface: Username is required` quando só a senha estava preenchida.
- (I) Nos envios com ambos os campos preenchidos por valores sintéticos inválidos — tanto com espaços nas extremidades quanto sem espaços — a interface exibiu `Epic sadface: Username and password do not match any user in this service`.
- (I) Após os envios observados, a URL permaneceu `https://www.saucedemo.com/`; não observei redirecionamentos.
- (I) Os campos preenchidos permaneceram com seus valores após a resposta. O campo de senha continuou mascarado na interface. A inspeção dos valores no DOM confirmou que os espaços nas extremidades continuavam presentes.
- (I) O console registrou respostas 401 e falhas de CORS/rede em requisições para endpoints de telemetria Backtrace. Não observei evidência de que essas falhas tenham impedido os envios ou alterado as respostas da interface de login.
- (R) Entradas compostas somente por espaços nos dois campos foram tratadas como credenciais não correspondentes, e não como campos vazios. Sem requisito disponível, não determinei se isso viola o comportamento esperado.

## Defeitos
1. **Possível defeito visual pendente de confirmação: mensagem de credenciais não correspondentes parcialmente cortada na interface.**  
   Onde: formulário da tela de login em `https://www.saucedemo.com/`.  
   Dados: credenciais sintéticas inválidas; também observado com espaços somente e valores inválidos com espaços nas extremidades.  
   O que aconteceu: a mensagem `Epic sadface: Username and password do not match any user in this service` apareceu quebrada em linhas e parcialmente encoberta/cortada nas capturas. O texto completo estava presente na árvore de acessibilidade.  
   Classificação: possível defeito visual pendente de confirmação; não foram fornecidos critérios visuais.  
   Reproduzido: aparência de corte observada nas capturas de três envios com a mensagem longa.  
   Evidências: [02 — campos com espaços em branco](02-whitespace-only.png), [03 — valores inválidos com espaços nas extremidades](03-whitespace-padded-invalid.png) e [04 — credenciais sintéticas inválidas](04-invalid-credentials.png).

## Perguntas
- Os campos contendo somente espaços devem ser tratados como vazios, ou a mensagem de credenciais não correspondentes é o comportamento esperado?

## Evidências
- [01 — envio com campos vazios](01-empty-submit.png)
- [02 — campos com espaços em branco](02-whitespace-only.png)
- [03 — valores inválidos com espaços nas extremidades](03-whitespace-padded-invalid.png)
- [04 — credenciais sintéticas inválidas](04-invalid-credentials.png)
- [05 — mensagem após a estabilização da página](05-stable-error.png)

## Não coberto
- Autenticação com credenciais válidas ou fluxos após login.
- Outros navegadores, dispositivos, tamanhos de tela ou condições de rede.
- Validação direta da API/servidor, acessibilidade completa e comparação com requisitos, que não foram fornecidos.
- Não foi determinado se as falhas de telemetria afetam outros comportamentos além dos observados.

## Dados de teste criados
- Nenhum registro foi criado. Foram inseridos apenas valores sintéticos nos campos do formulário durante os envios.

## Próximos charters
- Explorar apresentação, quebra de linha e leitura assistiva das diferentes mensagens de erro do login em tamanhos de viewport variados.
- Explorar regras de normalização de espaços em entradas de usuário e senha, após confirmar o comportamento esperado com o responsável pelo produto.

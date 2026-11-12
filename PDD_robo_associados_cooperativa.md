# PDD (rascunho): Robô de associados da cooperativa

> **Legenda:** o que consta no fluxo é **FATO**. Tudo marcado com **[A CONFIRMAR]** é suposição. Campos sem base no fluxo dizem *"não identificável no fluxo"*.
>
> **Aviso de segurança:** o JSON do fluxo contém credenciais em texto puro (chave de API na URL, usuário e senha de banco, string de conexão). Elas são citadas na seção 6, mas não são reproduzidas neste documento. Devem ser consideradas comprometidas e rotacionadas.

---

## 1. Identificação

| Campo | Conteúdo |
|---|---|
| Nome no fluxo | "Simulação CoE - Fluxo para revisão" (**FATO**) |
| Nome provável do processo | Sincronização/registro de valores por associado ativo em sistema de cooperativa **[A CONFIRMAR]** |
| Área provável | Não identificável no fluxo. A única pista é o domínio `api.exemplo-cooperativa.com` e a nota "robô de associados" (**FATO**). Hipótese: área de relacionamento com associados ou financeiro da cooperativa **[A CONFIRMAR]** |
| Dono do processo / demandante | Não identificável no fluxo |
| Natureza do fluxo | Exercício de simulação de CoE para revisão (**FATO**, conforme a nota adesiva). Não está claro se há um processo real por trás **[A CONFIRMAR]** |

## 2. Objetivo e justificativa

- **Objetivo (inferido do encadeamento):** buscar associados com status ativo e enviar, para cada um, um registro com nome e valor para outro endpoint **[A CONFIRMAR]**. O fluxo não declara a finalidade do "registro" nem o significado de "valor".
- **Justificativa:** não identificável no fluxo.
- **Observação de contexto:** a nota adesiva informa que o objetivo do exercício é revisar o fluxo com um checklist e abrir issues no GitHub, sem corrigir. Isso diz respeito à revisão, não ao objetivo de negócio do processo.

## 3. Processo atual, passo a passo

Sequência real das conexões (**FATO**):

| # | Nó n8n | O que faz tecnicamente (FATO) | Observação |
|---|---|---|---|
| 1 | `node` (Manual Trigger) | Início manual da execução | Nome genérico; não há agendamento nem gatilho automático |
| 2 | `HTTP Request1` | GET em `/associados?status=ativo` da API da cooperativa | A chave de API vai embutida na URL |
| 3 | `dados` (Set) | Define os campos `db_user`, `db_password` e `conn`, mantendo os demais campos | Os valores são fixos no nó |
| 4 | `HTTP Request2` | POST em `/registros` com `associado` = `$json.nome` e `valor` = `$json.valor` | Ver ressalva abaixo sobre a origem desses campos |
| 5 | `fazer` (Set) | Define o campo `x` = `"ok"` | Aparenta ser um marcador de fim; não valida o resultado do passo 4 |

**Pontos técnicos relevantes para o entendimento do processo:**

- Os campos `nome` e `valor` são lidos da saída do passo 3, que repassa os campos do passo 2 (`includeOtherFields: true`). Portanto só funcionam se a API de associados retornar campos com esses nomes **[A CONFIRMAR]**. O fluxo não mostra o formato da resposta.
- O nó `dados` define uma string de conexão com banco (`Server`, `Database`), mas **nenhum nó do fluxo usa um banco de dados**. Não é possível saber se é resíduo, preparação para um passo futuro ou outro uso **[A CONFIRMAR]**.
- O nó `fazer` não faz nada de negócio; só grava `x = ok`.
- Não há nó de erro, laço explícito, condição, notificação ou log.

## 4. Regras de negócio

**Identificadas no fluxo (FATO):**

- Só são consultados associados com `status=ativo`.

**Não identificáveis no fluxo:**

- Critério de cálculo ou origem do "valor".
- Regras de elegibilidade além do status ativo.
- Limites, validações, deduplicação, prazos ou condições de aprovação.

**Observação:** o filtro `status=ativo` é a única regra de negócio visível. Os demais itens (autenticação, set de variáveis, POST) são passos técnicos, não regras.

## 5. Exceções e tratamento

- **Tratamento de exceções no fluxo:** nenhum (**FATO**). Não há ramificação de erro, retentativa configurada, nó de tratamento de falhas nem alerta.
- Cenários sem tratamento visível, cujo comportamento desejado é **[A CONFIRMAR]**:
  - API de associados indisponível ou retornando erro/vazio.
  - Associado sem `nome` ou `valor`.
  - Falha no POST de um registro (o que acontece com os demais?).
  - Reexecução do fluxo: pode gerar registros duplicados? O fluxo não tem controle de idempotência.
  - Autenticação do POST: o `HTTP Request2` não mostra credencial nenhuma nos parâmetros.
- Tratamento manual atual (quem monitora, quem reprocessa): não identificável no fluxo.

## 6. Sistemas, dados e LGPD

**Sistemas (FATO):**

| Sistema | Uso |
|---|---|
| API `api.exemplo-cooperativa.com` (`/associados`) | Origem dos dados (leitura) |
| API `api.exemplo-cooperativa.com` (`/registros`) | Destino dos dados (escrita) |
| Servidor de banco em `10.0.0.9`, banco `coop` | Referenciado em variável, mas **não usado** por nenhum nó |
| n8n | Plataforma de orquestração |

**Dados pessoais e LGPD:**

- **Dado pessoal identificado:** nome do associado (`nome`), enviado ao endpoint `/registros` (**FATO**).
- **Possíveis dados pessoais adicionais** na resposta da API de associados, já que o fluxo repassa todos os campos adiante **[A CONFIRMAR]**: o fluxo não mostra o conteúdo, mas listas de associados costumam trazer CPF, contato, endereço etc.
- **Valor:** pode ser dado financeiro vinculado a pessoa identificada, o que o tornaria dado pessoal **[A CONFIRMAR]**.
- Base legal, finalidade, retenção, controlador/operador e compartilhamento: não identificáveis no fluxo.
- O uso de `includeOtherFields: true` sugere que dados desnecessários podem trafegar entre os nós (princípio da minimização) **[A CONFIRMAR]**.

**Segurança de credenciais (achados no fluxo, FATO):**

- Chave de API em texto puro na URL de uma consulta (fica exposta em logs, histórico de execução e controle de versão).
- Usuário e senha de banco e string de conexão em texto puro em um nó Set.
- **Recomendação:** tratar essas credenciais como vazadas, rotacioná-las e migrá-las para o gerenciador de credenciais do n8n antes de qualquer outro passo. Isso vale para qualquer repositório ou ambiente onde este JSON tenha sido compartilhado.

## 7. Volume e frequência

- **Frequência:** gatilho manual (**FATO**). Não há agendamento, então a periodicidade desejada é *não identificável no fluxo*.
- **Volume** (quantidade de associados, registros por execução, tempo de execução, picos): não identificável no fluxo.
- **Limites da API** (paginação, rate limit): não identificáveis. O fluxo não mostra paginação, então pode processar só a primeira página de resultados **[A CONFIRMAR]**.

## 8. Benefício esperado

Não identificável no fluxo. Hipóteses a validar com o demandante **[A CONFIRMAR]**: eliminação de lançamento manual de registros, redução de erros de digitação, ganho de tempo. Nenhum indicador (tempo por registro, volume, FTE liberado) pode ser derivado do fluxo.

---

## Lista de suposições

1. O processo registra, para cada associado ativo, um valor em outro sistema/endpoint.
2. A API de associados retorna os campos `nome` e `valor`.
3. O nó `fazer` marca o fim da execução e não tem função de negócio.
4. As credenciais de banco são resíduo de teste ou de um passo planejado.
5. O fluxo é um exercício de simulação de CoE e não necessariamente um processo em produção.
6. A área dona do processo é de relacionamento ou financeiro da cooperativa.

## Perguntas para o demandante

**Negócio**

1. Qual é o processo real? O que significa "registrar" um associado e qual o papel do "valor"?
2. Quem é o dono do processo e quem aprova o resultado?
3. De onde vem o valor e como ele é calculado?
4. Há outras regras além de "associado ativo" (carência, adimplência, tipo de associado)?

**Volume e operação**

5. Quantos associados ativos existem e com que frequência o robô deve rodar? Qual o horário ideal?
6. Hoje, quanto tempo e quantas pessoas o processo manual consome?
7. A API pagina resultados? Há limite de requisições?

**Exceções**

8. O que deve acontecer se um registro falhar: parar tudo, seguir e relatar, ou reprocessar depois?
9. Reexecutar o robô pode duplicar registros? Existe chave única no destino?
10. Quem deve ser avisado em caso de erro e por qual canal?

**Dados e LGPD**

11. Quais campos a API de associados retorna e quais são realmente necessários?
12. Qual a base legal e a finalidade do tratamento? Quem é o controlador e o operador?
13. Por quanto tempo os dados e os logs podem ser mantidos?

**Técnico e segurança**

14. A conexão de banco em `dados` é usada em algum lugar? Se não, pode ser removida?
15. Como o POST em `/registros` se autentica? As credenciais expostas já foram rotacionadas?
16. Qual o ambiente alvo (homologação ou produção) e há ambiente de teste?

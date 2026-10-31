# AGENTS.md — Guardrails de Desenvolvimento (Engenharia & Spec Kit)

Este arquivo define os **Guardrails de Desenvolvimento** (Always-On Harness) para todos os agentes de codificação que atuam neste repositório durante o desafio **Caqui Invest**.

> **Escopo:** Estes guardrails governam **como nós desenvolvemos, especificamos, testamos e versionamos o projeto** (e não a lógica interna de runtime da aplicação).

---

## 1. Subagentes Guardiões do Processo de Desenvolvimento

Antes de concluir qualquer etapa do ciclo Spec Kit (`specify` → `plan` → `tasks` → `implement`), o agente principal DEVE acionar os subagentes de auditoria de engenharia abaixo:

| Subagente (`TypeName`) | Foco no Desenvolvimento (Engenharia) | O que Audita / Bloqueia no Repositório |
| :--- | :--- | :--- |
| **`guardrail-dev-spec-entregaveis`** | **Governança Spec-First e Entregáveis do Desafio** | Bloqueia escrita de código antes da Spec estar aprovada; audita se `SPEC.md` (raiz) responde às 6 perguntas com contas à mostra e tabela de decisões; verifica sincronia com `.specify/` e presença da seção `Como usamos IA` (`templates/COMO-USAMOS-IA.md`) no `README.md`. |
| **`guardrail-dev-seguranca-dados`** | **Segurança do Repositório, Segredos e Dados Sintéticos** | Inspeciona código, testes, logs e `.gitignore` para garantir: zero vazamento de chaves de API/tokens, uso exclusivo dos dados sintéticos de `dados/` (zero PII real, nem dos próprios participantes), zero imitação de marca bancária real e sanitização de CPF em logs de desenvolvimento. |
| **`guardrail-dev-qualidade-testes`** | **Determinismo de Código, Ingestão de Dados e Validação** | Audita o código para garantir que cálculos financeiros/datas usem funções determinísticas testáveis (sem delegar matemática ao LLM), que o loader de dados filtre artigos obsoletos (`KB-007.md` de 2024) e use o marco fixo `2026-11-18T10:00:00-03:00`, e audita o resultado de `ferramentas/checar_respostas.py` + leitura semântica contra `resposta_esperada`. |

---

## 2. Regras Inegociáveis de Engenharia (Durante o Desenvolvimento)

1. **Proibido Codificar Antes da Spec (Spec-Driven Development):**
   - Nenhuma linha de código de aplicação (`src/`, `app/`, etc.) pode ser criada antes de preencher e validar o `SPEC.md` do desafio e o `specs/<feature>/spec.md` do Spec Kit.
   - Qualquer mudança de escopo ou decisão arquitetural durante o dia deve ser registrada imediatamente na seção `## Decisões` do `SPEC.md`.

2. **Higiene de Segredos e Dados no Código e Git:**
   - **Zero Segredos:** Nunca escreva chaves de API, tokens ou credenciais em arquivos versionados. Use variáveis de ambiente (`.env` ignorado no `.gitignore`) e forneça `.env.example` vazio.
   - **100% Dados Sintéticos:** Nunca crie mocks ou testes com nomes, CPFs, e-mails ou telefones reais (nem mesmo dos integrantes do grupo). Use estritamente os registros de `dados/clientes.json`, `posicoes.csv`, `operacoes.csv` e `conversas.csv`.
   - **Identidade Fictícia:** Nunca utilize nomes, logotipos ou referências a bancos/corretoras reais na UI ou nos textos; a marca é exclusivamente **Caqui Invest**.

3. **Arquitetura Determinística vs. IA Generativa:**
   - É proibido construir código que faça chamadas à internet para buscar CDI, Selic, Poupança ou calendário de feriados. O código deve ler exclusivamente `dados/taxas.json` e `dados/calendario.json`.
   - É proibido deixar o LLM calcular prazos de dias úteis, alíquotas de IR regressivo ou comparação de rentabilidade no "olhômetro". O desenvolvedor/agente deve escrever código determinístico para isso e cobrir com testes automatizados.
   - O indexador/loader de `dados/base_conhecimento/` desenvolvido no código deve obrigatoriamente validar o metadado `revisado_em` ou bloquear artigos substituídos (`KB-007.md`).

4. **Verificação Obrigatória ("Verificado na Mão" + Script):**
   - Toda regra de negócio declarada como `"Trata = Sim"` no `SPEC.md` deve ser transformada em teste executável via `python ferramentas/checar_respostas.py respostas.jsonl`.
   - Como exige o desafio: *"Passar nas pistas não prova que a resposta está certa. Leiam as respostas."* O subagente `guardrail-dev-qualidade-testes` deve inspecionar o texto completo gerado em `respostas.jsonl` comparando com `resposta_esperada` e registrar no `README.md` (`Onde a IA errou`) qualquer erro detectado durante o desenvolvimento.

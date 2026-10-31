---
name: guardrail-dev-qualidade-testes
description: >-
  Guardrail de desenvolvimento para auditar a arquitetura determinística do código,
  higiene na ingestão de dados (bloqueio do KB-007 obsoleto, uso de taxas.json e calendario.json,
  marco temporal 18/11/2026 10h) e execução/auditoria semântica do checar_respostas.py.
---

# Guardrail de Desenvolvimento: Qualidade de Código, Determinismo e Testes

Use esta skill durante as fases de planejamento (`plan`), tarefas (`tasks`) e implementação (`implement`) para garantir que o código produzido não cometa as falhas clássicas de engenharia com IA apontadas no case.

## Checklist de Auditoria Técnica de Código

1. **Separação Estrita: Motor Determinístico vs. Camada de Linguagem:**
   - **Proibido no código:** Pedir para um LLM fazer conta de dias úteis, verificar se uma data é feriado, decidir qual faixa da tabela regressiva de IR usar ou calcular juros compostos da poupança/CDB.
   - **Exigido no código:** Funções puras e determinísticas que leiam `dados/calendario.json`, `dados/taxas.json`, `dados/posicoes.csv` e `dados/operacoes.csv` usando a constante de referência `NOW = "2026-11-18T10:00:00-03:00"` (com suporte a override por `contexto.momento` para testes).
2. **Auditoria de Ingestão da Base de Conhecimento (Data Hygiene):**
   - Verifique no código do loader de `dados/base_conhecimento/` se há filtro explícito para ignorar artigos obsoletos/substituídos — especificamente `KB-007.md` (`revisado_em: 2024-03-12`, que contém o horário de corte antigo de 15h), garantindo que apenas `KB-006.md` (`revisado_em: 2026-10-01`, corte às 16h) seja indexado.
   - Verifique que o código não realiza buscas web ou chamadas a APIs externas de cotações; todas as taxas devem vir de `dados/taxas.json`.
3. **Execução e Auditoria Dupla de Testes (`checar_respostas.py`):**
   - Passo 1 (Automatizado): Executar `python ferramentas/checar_respostas.py respostas.jsonl` (com `--regras` correspondentes ao escopo da Spec ou todas as regras) e exigir exit code `0`.
   - Passo 2 (Auditoria Semântica "Verificado na Mão"): Ler linha a linha o arquivo `respostas.jsonl` gerado e comparar com `resposta_esperada` em `dados/perguntas_teste.jsonl`, verificando se uma resposta passou nas pistas (`deve_conter` / `nao_deve_conter`) mas contém lógica incorreta, tom inadequado ou omissão de informação essencial.

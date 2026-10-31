---
name: guardrail-dev-spec-entregaveis
description: >-
  Guardrail de desenvolvimento para auditar a governança Spec-Driven Development (Spec Kit)
  e os entregáveis obrigatórios do desafio Caqui Invest (SPEC.md na raiz com as 6 perguntas,
  log de decisões, alinhamento spec/plan/tasks e seção Como usamos IA no README.md).
---

# Guardrail de Desenvolvimento: Governança Spec-First e Entregáveis

Use esta skill para auditar o **processo de desenvolvimento** do projeto antes de escrever código e antes do deploy final.

## Checklist de Auditoria de Engenharia

1. **Trava Spec-First ("Antes de qualquer código"):**
   - Verifique se `SPEC.md` (baseado em `templates/SPEC.md`) e `specs/<feature>/spec.md` existem e estão preenchidos antes de permitir a criação de arquivos de código-fonte.
2. **Auditoria das 6 Perguntas Obrigatórias no `SPEC.md`:**
   - **Pergunta 1 (Qual parte do problema?):** Cita números concretos do `CASE.md` ou de `dados/conversas.csv`?
   - **Pergunta 2 (Qual alavanca?):** Define claramente entre evitar contato, resolver sem humano ou fazer o humano render mais?
   - **Pergunta 3 (Quanto isso move?):** Possui a tabela de premissas (`Premissa | Valor | De onde veio`) e a **conta matemática aberta** demonstrando horas economizadas, redução de recontato ou ganho de capacidade?
   - **Pergunta 4 (Como a confiança aparece?):** Descreve os elementos visuais de prova (fonte `KB-XXX`, dados da conta, status) e onde fica a saída para humano?
   - **Pergunta 5 (O que fica de fora?):** Explicita o corte de escopo para viabilizar a entrega no mesmo dia?
   - **Pergunta 6 (Quais regras do case trata?):** A tabela das 12 regras está preenchida indicando `Trata?`, `Como vira teste` e `Se não trata, por quê`? Se a solução não conversar com o cliente (ex.: ferramenta puramente interna), justificou os itens 1, 3, 4 e 5 de *"O que vale para qualquer caminho"* (lembrando que 2 e 6 valem sempre)?
3. **Rastreabilidade de Decisões e Erros da IA:**
   - A tabela `## Decisões` no `SPEC.md` está sendo atualizada conforme o projeto evolui?
   - O `README.md` contém a seção `## Como usamos IA` (`templates/COMO-USAMOS-IA.md`) documentando **Ferramentas**, **Harness**, **Verificado na mão** e **Onde a IA errou**, além do link da URL pública no topo?

---
name: guardrail-runtime-triagem-seguranca
description: >-
  Guardrail de runtime para auditar a triagem síncrona pré-LLM, priorização incondicional
  (Golpe > Vulnerável > Prompt Injection > Recomendação), bloqueio preventivo de sessão
  e isolamento de dados de sessão (prevenção a BOLA).
---

# Guardrail de Runtime: Triagem Prévia, Prevenção a Fraudes e Isolamento

Use esta skill para auditar a implementação do pipeline de entrada do assistente antes do envio de prompts ao LLM.

## Checklist de Auditoria do Pipeline de Entrada

1. **Precedência Incondicional da Triagem:**
   - Verifique se a função de triagem (`triar` ou similar) executa antes de qualquer processador de intenção ou chamada ao LLM.
   - Verifique se o sinal de **Golpe / Fraude** (Regra 5, KB-019/KB-020) interrompe imediatamente o fluxo, mesmo que o usuário pergunte sobre prazo ou saldo na mesma frase (cenário PT-19).
   - Verifique se casos de **Cliente Vulnerável** (Regra 6, KB-028: idade >= 60, pressão de parente ou terceiro) recebem encaminhamento prioritário a humano sem confirmação de resgate.
   - Verifique se pedidos de **Recomendação** (Regra 4, KB-029) e tentativas de roleplay ("aja como meu consultor") são interceptados sem indicação de produto.

2. **Isolamento de Sessão e Anti-Prompt Injection:**
   - Verifique se o identificador do cliente provém unicamente do token/sessão HTTP autenticado.
   - Verifique se referências a outros clientes no texto (ex: `C-0042`, CPFs) são ignoradas para efeito de busca de dados (mitigação de Broken Object Level Authorization).
   - Verifique se a mensagem do usuário é delimitada no prompt como dado passivo (ex: `<mensagem_cliente>`), nunca como instrução do sistema.

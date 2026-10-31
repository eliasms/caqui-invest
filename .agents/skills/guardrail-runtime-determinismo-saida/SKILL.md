---
name: guardrail-runtime-determinismo-saida
description: >-
  Guardrail de runtime para auditar a validação determinística da resposta pós-LLM,
  extração de fatos em R$ e datas, bloqueio de alucinações e palavras proibidas,
  e fallback gracioso por templates.
---

# Guardrail de Runtime: Determinismo de Saída, Validação e Fallback

Use esta skill para auditar os módulos de resposta, o validador pós-LLM e os mecanismos de contingência do assistente.

## Checklist de Auditoria de Saída e Fallback

1. **Validação de Fatos no Texto Gerado:**
   - Verifique se o código possui um verificador de saída que analisa a resposta do LLM antes da exibição ao usuário.
   - Verifique se datas, percentuais e valores monetários expressos em R$ no texto gerado são comparados contra a estrutura de fatos determinísticos provida pelo backend.
   - Se um valor ou data divergente for detectado, garanta que a resposta seja descartada e substituída por template determinístico.

2. **Filtro de Linguagem e Termos Alarmistas:**
   - Verifique se a resposta gerada é inspecionada para bloquear termos proibidos: `"sumiu"`, `"perdido"`, `"extraviado"`, ou expressões de recomendação (`"recomendo"`, `"melhor opção"`).

3. **Fallback Resiliente por Template:**
   - Verifique se o sistema opera normalmente com `LLM_ENABLED=false` ou na ausência de chaves de API, respondendo com precisão determinística e fontes completas via templates.
   - Verifique se as fontes citadas (`KB-XXX`, `OP-XXXXX`, `P-XXXX`) são injetadas estruturadamente pelo backend a partir dos fatos, nunca delegadas à redação do modelo.

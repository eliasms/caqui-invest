---
name: guardrail-dev-seguranca-dados
description: >-
  Guardrail de desenvolvimento para auditar segurança do repositório, prevenção de vazamento
  de segredos/chaves de API, uso exclusivo de dados sintéticos (zero PII real), não-imitação
  de marcas reais e sanitização de logs no código.
---

# Guardrail de Desenvolvimento: Segurança do Repositório e Dados Sintéticos

Use esta skill para inspecionar arquivos de código, configuração, testes e logs criados durante o desenvolvimento.

## Checklist de Auditoria de Repositório e Código

1. **Prevenção de Vazamento de Segredos:**
   - Verifique com busca/grep se não há nenhuma chave de API (`sk-`, `AIza`, tokens, senhas ou credenciais) hardcoded no código ou em arquivos de configuração.
   - Confirme que `.gitignore` bloqueia `.env`, `.env.local`, `.env.*` e diretórios temporários.
2. **Garantia de Dados 100% Sintéticos (Regra Universal 6):**
   - Confirme que todos os testes, fixtures, seeds e exemplos na interface usam **apenas** os dados sintéticos fornecidos em `dados/` (`C-0001` a `C-0050`).
   - Bloqueie imediatamente qualquer tentativa de usar nome, CPF, telefone ou e-mail real (mesmo que seja de um participante da equipe).
3. **Proteção de PII em Logs e Console:**
   - Inspecione chamadas de `console.log`, `print` ou `logger` no código desenvolvido para garantir que CPFs nunca sejam reconstruídos ou logados sem máscara (`***.***.XXX-**`) e que senhas jamais sejam solicitadas ou armazenadas.
4. **Identidade Visual Fictícia:**
   - Verifique se o front-end e os textos mencionam exclusivamente a **Caqui Invest**, sem copiar nomes, logos ou marcas de instituições financeiras reais.
5. **Isolamento de Input no Código:**
   - Verifique se o código trata entradas de texto do usuário como dados parametrizados (sem `eval`, sem concatenação insegura que permita sobrescrever diretivas de sistema ou acessar registros de outro `id_cliente` sem validação de sessão).

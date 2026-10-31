# AGENTS.md — Guardrails de Desenvolvimento e Runtime (Harness do Agente)

Este arquivo define os **Guardrails de Desenvolvimento e de Runtime** (Always-On Harness) para todos os agentes de codificação e auditoria que atuam neste repositório durante o desafio **Caqui Invest**.

> **Escopo Amplo:** Este documento governa tanto **como nós desenvolvemos, especificamos e testamos** (Dev Harness) quanto **como o chatbot se comporta em tempo de execução** para garantir segurança, conformidade e precisão matemática (Runtime Guardrails).

---

## 1. Subagentes Guardiões do Processo de Engenharia

Antes de concluir qualquer etapa do ciclo Spec Kit (`specify` → `plan` → `tasks` → `implement`), o agente principal DEVE acionar os subagentes de auditoria de engenharia abaixo:

| Subagente (`TypeName`) | Foco no Desenvolvimento (Engenharia) | O que Audita / Bloqueia no Repositório |
| :--- | :--- | :--- |
| **`guardrail-dev-spec-entregaveis`** | **Governança Spec-First e Entregáveis do Desafio** | Bloqueia escrita de código antes da Spec estar aprovada; audita se `SPEC.md` (raiz) responde às 6 perguntas com contas à mostra e tabela de decisões; verifica sincronia com `.specify/` e presença da seção `Como usamos IA` (`templates/COMO-USAMOS-IA.md`) no `README.md`. |
| **`guardrail-dev-seguranca-dados`** | **Segurança do Repositório, Segredos e Dados Sintéticos** | Inspeciona código, testes, logs e `.gitignore` para garantir: zero vazamento de chaves de API/tokens, uso exclusivo dos dados sintéticos de `dados/` (zero PII real, nem dos próprios participantes), zero imitação de marca bancária real e sanitização de CPF em logs de desenvolvimento. |
| **`guardrail-dev-qualidade-testes`** | **Determinismo de Código, Ingestão de Dados e Validação** | Audita o código para garantir que cálculos financeiros/datas usem funções determinísticas testáveis (sem delegar matemática ao LLM), que o loader de dados filtre artigos obsoletos (`KB-007.md` de 2024) e use o marco fixo `2026-11-18T10:00:00-03:00`, e audita o resultado de `ferramentas/checar_respostas.py` + leitura semântica contra `resposta_esperada`. |
| **`guardrail-runtime-triagem-seguranca`** | **Defesa Pré-LLM (Triagem e Engenharia Social)** | Garante que o pipeline de execução possua um interceptador síncrono antes do LLM com prioridade: Golpe > Vulnerável > Prompt Injection > Recomendação. Bloqueia respostas de prazo misturadas com golpe. |
| **`guardrail-runtime-determinismo-saida`** | **Defesa Pós-LLM (Validação de Fatos e Anti-Alucinação)** | Garante que exista um validador de saída que extrai datas e valores em R$ da resposta do LLM, compara com os fatos determinísticos providos pelo backend e aciona fallback por template se houver divergência ou termos proibidos. |

---

## 2. Regras Inegociáveis de Engenharia (Durante o Desenvolvimento)

1. **Proibido Codificar Antes da Spec (Spec-Driven Development):**
   - Nenhuma linha de código de aplicação (`apps/backend/`, `apps/web/`, etc.) pode ser criada antes de preencher e validar o `SPEC.md` do desafio e o `specs/001-onde-esta-meu-dinheiro/spec.md` do Spec Kit.
   - Qualquer mudança de escopo ou decisão arquitetural durante o dia deve ser registrada imediatamente na seção `## Decisões` do `SPEC.md`.

2. **Higiene de Segredos e Dados no Código e Git:**
   - **Zero Segredos:** Nunca escreva chaves de API, tokens ou credenciais em arquivos versionados. Use variáveis de ambiente (`.env` ignorado no `.gitignore`) e forneça `.env.example` vazio. A chave do LLM existe exclusivamente no backend.
   - **100% Dados Sintéticos:** Nunca crie mocks ou testes com nomes, CPFs, e-mails ou telefones reais (nem mesmo dos integrantes do grupo). Use estritamente os registros de `dados/clientes.json`, `posicoes.csv`, `operacoes.csv` e `conversas.csv`.
   - **Identidade Fictícia:** Nunca utilize nomes, logotipos ou referências a bancos/corretoras reais na UI ou nos textos; a marca é exclusivamente **Caqui Invest**.

3. **Arquitetura Determinística vs. IA Generativa:**
   - É proibido construir código que faça chamadas à internet para buscar CDI, Selic, Poupança ou calendário de feriados. O código deve ler exclusivamente `dados/taxas.json` e `dados/calendario.json`.
   - É proibido deixar o LLM calcular prazos de dias úteis, alíquotas de IR regressivo ou comparação de rentabilidade. O desenvolvedor/agente deve escrever código determinístico para isso e cobrir com testes automatizados.
   - O indexador/loader de `dados/base_conhecimento/` desenvolvido no código deve obrigatoriamente bloquear artigos substituídos (`KB-007.md`).

4. **Verificação Obrigatória ("Verificado na Mão" + Script):**
   - Toda regra de negócio declarada como `"Trata = Sim"` no `SPEC.md` deve ser transformada em teste executável via `python ferramentas/checar_respostas.py respostas.jsonl`.
   - O agente e desenvolvedor devem inspecionar o texto completo gerado em `respostas.jsonl` comparando com `resposta_esperada` e registrar no `README.md` (`Onde a IA errou`) qualquer erro detectado.

---

## 3. Guardrails de Runtime e Arquitetura do Chatbot (Proteções em Execução)

1. **Pre-Execution Gate: Pipeline de Triagem Determinística:**
   - Toda mensagem recebida DEVE ser processada pelo módulo de triagem antes de qualquer inferência de LLM.
   - A triagem possui precedência incondicional com a ordem:
     `Golpe (Regra 5) > Cliente Vulnerável (Regra 6) > Injeção de Prompt (Regra 11) > Recomendação (Regra 4) > Negócio`.
   - Caso `PT-19` (sinal de golpe misturado com dúvida legítima de prazo): interrompe o fluxo de prazo imediatamente, emite alerta de segurança (`KB-019`/`KB-020`), simula bloqueio de sessão e encaminha para segurança 24/7 sem fila.

2. **Post-Execution Gate: Validador de Saída e Fallback por Template:**
   - O backend inspeciona a redação do LLM antes de entregá-la ao cliente.
   - Extrator regex valida se todas as datas e valores em R$ redigidos constam nos fatos determinísticos entregues no contexto.
   - Palavras proibidas (`"sumiu"`, `"perdido"`, `"recomendo"`, `"melhor opção"`) causam rejeição do texto e acionamento automático do **template determinístico formatado**.

3. **Isolamento de Sessão e Prevenção Ativa de BOLA (Broken Object Level Authorization):**
   - A identidade do cliente decorre exclusivamente da sessão/token HTTP (`login simulado`).
   - Identificadores de clientes (`C-0042`), CPFs ou nomes citados dentro do texto da mensagem do usuário são **completamente ignorados** para acesso a dados, evitando personificação via prompt injection.
   - O texto do usuário entra no prompt delimitado categoricamente como `<dado_usuario>`, nunca como comando.

4. **Anti-Doom Loop e Saída para Atendente Humano (Decreto SAC):**
   - O botão "Falar com uma pessoa" permanece visível e funcional em qualquer tela.
   - Teto máximo de **5 passos** de conversa por atendimento.
   - Limite de **2 incompreensões consecutivas** antes de ofertar proativamente o encaminhamento humano.
   - Horário das 08h às 20h transfere para humano; fora de horário registra protocolo (`CQ-AAAAMMDD-NNNN`) e prazo para retorno às 08h. Toda transferência entrega histórico, resumo e protocolo sem o cliente repetir a demanda.

5. **Proteção de Custo e Circuit Breaker na URL Pública:**
   - Rate limiting por IP e por sessão ativa no endpoint de chat.
   - Tamanho máximo estrito de mensagem de entrada para evitar estouro de tokens.
   - **`LLM_ENABLED=false`:** Chave no `.env` que ativa o modo degradação graciosa com respostas 100% determinísticas por templates em caso de esgotamento de quota ou indisponibilidade da API.

---

## 4. Harness de Execução por Sessões (Roadmaps do Projeto)

Para trabalhar de forma iterativa, o agente deve seguir a sequência estabelecida nos roadmaps da raiz:

1. **MVP0 ([`ROADMAP-MVP0.md`](ROADMAP-MVP0.md)):** Prova de conceito mínima: "quanto eu tenho aplicado?" (saldo e posições de Marta C-0001 com fontes `P-0001`, `P-0002` e `C-0001`), separação estrita de cálculo em código e texto redigido, com fallback de template.
2. **MVP ([`ROADMAP-MVP.md`](ROADMAP-MVP.md)):** Núcleo completo do assistente "Onde está meu dinheiro?" (Prazos, status e liquidação com corte das 16h, feriados, posições, triagem de golpe e recomendação, avaliação de 40 perguntas via `checar_respostas.py`).
3. **Versão Completa ([`ROADMAP.md`](ROADMAP.md)):** Monorepo completo com `apps/backend` (Fastify/Node 22), `apps/web` (React/Vite), tela do atendente humano com protocolos e histórico, rate limiting de produção (T23) e deploy em URL pública.

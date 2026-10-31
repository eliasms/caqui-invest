<!--
Sync Impact Report
- Version change: 1.0.0 → 1.1.0
- Modified principles:
  - Princípio I: Adicionado "Confiança antes de abrangência" e tratamento estrito de fora de escopo.
  - Princípio II: Fortalecido com regras de anti-doom loop (máx 2 erros, máx 5 turnos) e protocolo rastreável.
  - Princípio III: Estabelecida a hierarquia incondicional de triagem prévia (Golpe > Vulnerável > Injeção > Recomendação).
  - Princípio IV: Adicionada a regra inegociável de validação de saída pós-LLM (verificador de fatos R$ e datas) e fallback de template.
  - Princípio V: Blindagem ativa contra quebra de sessão (ignorar IDs/CPFs no texto da mensagem contra Broken Object Level Authorization).
- Added sections:
  - Restrições de Arquitetura, Operação e Proteção de Custo na URL Pública (Rate Limiting, Circuit Breaker LLM_ENABLED=false, Chave só no Backend).
  - Regras de Borda de Liquidação (Corte 16h00 inclusive vs 16h01 pós-corte).
-->
# Caqui Invest Constitution

## Core Principles

### I. Confiança Visível e Rastreabilidade de Fontes (NON-NEGOTIABLE)
- **Confiança antes de abrangência:** O assistente resolve com excelência um conjunto delimitado de assuntos ("onde está meu dinheiro": status, prazos de liquidação, posições e saldo) em vez de arriscar alucinações tentando responder tudo.
- Perguntas fora de escopo (rentabilidade futura, IR complexo, produtos que a Caqui não oferece) DEVEM receber recusa explícita e educada, informando a limitação e oferecendo especialista/humano, nunca uma resposta improvisada.
- **Rastreabilidade obrigatória:** Toda informação factual apresentada DEVE exibir explicitamente sua origem: o código do artigo atualizado da base de conhecimento (`KB-001` a `KB-030`, com exclusão obrigatória de artigos obsoletos como `KB-007`) ou o identificador exato da conta/operação (`posicoes.csv`, `operacoes.csv`, `taxas.json`, `calendario.json`).
- Se um prazo, taxa ou produto não constar na base oficial (`dados/`), o sistema DEVE declarar que não sabe (Regra 9) e oferecer humano. É proibido inventar dados ou buscar taxas externas na internet.

### II. Saída Humana Sem Repetição e Prevenção de Doom Loop (Conformidade Decreto SAC 13.135/2026)
- O acesso a um atendente humano DEVE estar visível e acessível em qualquer tela ou passo da jornada.
- **Horário de Atendimento:**
  - Das 08h às 20h (BRT, 7 dias por semana): encaminhamento direto para humano com tempo estimado de espera e sem menus labirínticos.
  - Fora desse horário (20h às 08h): emissão imediata de protocolo determinístico (`CQ-AAAAMMDD-NNNN`) e aviso claro de que o retorno da equipe humana ocorrerá a partir das 08h (Regra 12).
- **Anti-Doom Loop (Regra 8):** Limite máximo de **5 passos/turnos** até resolver ou transferir para humano. Após **2 respostas consecutivas** sem compreender a intenção do cliente, o assistente DEVE ofertar proativamente o atendimento humano.
- **Transição com Contexto:** Em qualquer encaminhamento humano ou de segurança, o histórico completo da conversa, resumo do caso e número de protocolo DEVEM ser repassados, garantindo que o cliente **nunca repita a demanda**.

### III. Triagem Crítica com Precedência Incondicional (NON-NEGOTIABLE)
Toda mensagem passa por triagem determinística antes de qualquer chamada ao LLM. A triagem prevalece incondicionalmente sobre qualquer dúvida de negócio secundária na mesma mensagem (ex.: dúvida de prazo combinada com relato de engenharia social).
Hierarquia de prioridade na triagem:
1. **Golpe / Engenharia Social (Regra 5):** Sinais de contato falso de gerente/segurança, pedido de senha, código SMS, Pix de teste ou instalação de software de suporte remoto → interrompe imediatamente qualquer outro assunto, orienta o cliente (`KB-019`/`KB-020`), aciona bloqueio preventivo simulado de sessão e transfere para a equipe de segurança 24/7 sem fila (`seguranca_imediata`).
2. **Cliente Vulnerável (Regra 6):** Pessoa idosa (60+ anos), sinais de confusão mental ou relato de pressão de terceiros/familiares para resgate total (`KB-028`) → acolhimento prioritário, lembrete de que transferências só ocorrem para mesma titularidade e transferência prioritária para humano (`encaminhar_humano`).
3. **Instrução Escondida / Prompt Injection (Regra 11):** Tentativas de burlar regras, comando de administrador ou troca de persona → o texto do cliente é tratado estritamente como dado não confiável, sendo a instrução ignorada/recusada sem alterar o comportamento das regras.
4. **Zero Recomendação (Regra 4):** O sistema NUNCA DEVE indicar produto, alocação percentual ou "melhor opção" de investimento, mesmo sob insistência ou roleplay. Explica conceitos de forma neutra e encaminha para especialista humano (`KB-029`).

### IV. Fato Determinístico, Texto Generativo e Validação Pós-LLM (NON-NEGOTIABLE)
- Cálculos de liquidação, prazos úteis, tabela de IR regressivo, saldos e valores NUNCA DEVEM ser calculados pelo LLM. Todos os fatos matemáticos e temporais DEVEM ser gerados deterministicamente pelo código backend a partir de `dados/`.
- **Marco Temporal ("Agora"):** Padrão `2026-11-18T10:00:00-03:00`, ajustável na interface para demonstrar cortes e horários.
- **Regras de Corte e Liquidação (Regra 1 e KB-006):**
  - Corte às 16h em dias úteis segundo `calendario.json`. Pedidos até `16:00:00` contam no mesmo dia útil; a partir de `16:00:01` ou fins de semana/feriados, o dia-base (D0) é o próximo dia útil.
  - CDB/LCI liquidez diária: D+0; Fundo RF FIF: D+1 útil (cotização) e D+2 úteis (pagamento); Fundo MM FIF: D+30 corridos (ajustado para útil) + 2 úteis para pagamento; Tesouro Selic: D+1 útil; Ações: D+2 úteis.
  - Operações com `liquidacao_prevista` vazia no CSV DEVEM ser calculadas deterministricamente pela regra de prazos, nunca estimadas.
- **Validador de Saída (Post-Processor Gate):** O texto redigido pelo LLM passa por inspeção antes de ir ao cliente. Datas e valores em R$ presentes no texto gerado DEVEM bater exatamente com os fatos fornecidos. Expressões alarmistas ("sumiu", "perdido") ou recomendações disfarçadas causam descarte imediato da resposta generativa e ativação do **fallback determinístico por template**.

### V. Privacidade, Isolamento de Dados e Mitigação Ativa de Quebra de Sessão (NON-NEGOTIABLE)
- **Identificação Unicamente por Sessão (Regra 10):** O cliente é identificado pela sessão/token HTTP autenticado.
- **Defesa Ativa contra Broken Object Level Authorization (BOLA):** Identificadores de clientes (`C-XXXX`), nomes ou CPFs citados dentro do texto da mensagem do usuário DEVEM ser sumariamente desconsiderados para acesso a dados. Um cliente jamais acessa informações de terceiros, mesmo alegando procuração ou parentesco.
- **Máscara de PII e Zero Senhas:** CPFs aparecem sempre mascarados (`***.***.XXX-**`) na tela e nos logs de backend. O sistema nunca solicita nem aceita senha.
- **Dados 100% Sintéticos:** Exclusivo uso dos arquivos sintéticos de `dados/`. Zero dados de pessoas reais em código, testes ou apresentações.

---

## Restrições de Arquitetura, Operação e Proteção de Custo na URL Pública

1. **Separação Front e Backend:** Duas aplicações isoladas. A chave do LLM reside **exclusivamente no backend** (`.env` fora do git). O front consome apenas a API local/servida.
2. **Proteção contra Custos e Abuso na URL Pública:**
   - Rate limiting por IP e por sessão ativa no endpoint de chat.
   - Tamanho máximo estrito para as mensagens recebidas do usuário.
   - **Circuit Breaker (`LLM_ENABLED=false`):** Interruptor de emergência no `.env` que desativa chamadas externas ao LLM e opera o sistema com templates determinísticos sem derrubar a aplicação nem reprovar os testes.
3. **Logger Sanitizado:** Interceptador de log que expurga CPFs não mascarados e mensagens cruas antes de gravar em disco ou observabilidade.

---

## Regras de Negócio e Restrições de Domínio (As 12 Regras da Caqui)

1. **Direito de Cancelamento e Reclamação Sem Atrito (Regra 7):** Encerramento de conta é atendido na primeira resposta (máximo 1 retenção leve), orientando resgate e transferência (`KB-027`). Reclamação gera protocolo imediato com prazo de até 7 dias corridos (`KB-026`).
2. **Desambiguação Obrigatória:** Quando o cliente tiver mais de uma operação pendente parecida, o assistente lista as opções e solicita a confirmação de qual operação deseja consultar, proibido de adivinhar.
3. **Sincronização com o Case:** O arquivo `SPEC.md` na raiz deve ser mantido atualizado respondendo às 6 perguntas obrigatórias do desafio, e o `README.md` deve conter a seção `Como usamos IA` e a URL pública no topo.

---

## Fluxo de Desenvolvimento e Portões de Qualidade (Spec-Driven & Test-First)

1. **Fluxo Spec Kit:** Toda funcionalidade segue `/speckit-constitution` → `/speckit-specify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`.
2. **Portão Automático Obrigatório (`checar_respostas.py`):** Validação das perguntas de teste executando:
   `python ferramentas/checar_respostas.py respostas.jsonl --regras <escopo>`
   Critério de aceite: 100% de aprovação nas pistas automáticas e conformidade semântica com `resposta_esperada`.
3. **Auditoria Humana ("Verificado na Mão"):** Respostas geradas e relatórios de execução devem ser lidos e comparados com a base de conhecimento e regras de negócio antes de qualquer publicação.

---

## Governance

Esta Constituição prevalece sobre qualquer especificação, plano ou tarefa. Modificações exigem registro de Sync Impact Report e incremento semântico de versão.

**Version**: 1.1.0 | **Ratified**: 2026-10-31 | **Last Amended**: 2026-10-31

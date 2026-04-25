# Matriz de Auditoria de Confiabilidade - Bio Storage v1

## 1. Objetivo
Este documento define a Matriz de Auditoria para os Gates G0 (Formalizacao), G1 (In-silico), G2 (Heterogeneidade), G3 (Fault Injection Adversarial) e G4 (Longitudinal) do projeto Sanguine Ledger.

O foco principal e demonstrar, com evidencia auditavel:

- Safety: limite superior para erro nao detectado de saida aceita.
- Liveness: limite inferior para disponibilidade efetiva no horizonte de missao.
- Fail-stop preferencial: erro invalido deve tender a abort, nao a accept.
- Dissipacao de causa comum: reducao mensuravel de `beta_shared` e impacto em `N_eff`.

## 2. Objetivo Formal e Variaveis de Governanca

### 2.1 Objetivo formal (fonte de verdade v8.0)

- Safety:

  $\Pr(\hat m \neq m \land accept) \le \epsilon_{UE}$

- Liveness:

  $\Pr(\exists t \le T_*: accept) \ge 1 - \epsilon_{live}$

- Fail-stop preferencial:

  $\Pr(\hat m \neq m \land accept) \ll \Pr(\hat m = m \land abort)$

- Risco total:

  $P_{UE}(T)=\sup_{\theta\in\Theta}\Pr_{\theta}(\hat m\neq m \land c=accept,\ t\le T)$

- Confiabilidade de missao:

  $R(T)=1-P_{UE}(T)-P_{unavail}(T)$

### 2.2 Decomposicao de risco por dominio

$$
P_{UE}(T) \le
\epsilon_{ch}+\epsilon_{sync}+\epsilon_{ver}+\epsilon_{cons}+\epsilon_{key}+\epsilon_{ops}+\epsilon_{adv}
$$

Regra de fechamento:

$$
\sum_i \epsilon_i \le \epsilon_{target}
$$

### 2.3 Causa comum e correlacao

$$
\beta_{shared}=\beta_{substrato}+\beta_{amostra}+\beta_{pipeline}+\beta_{modelo}+\beta_{chave}+\beta_{adversarial}
$$

Procedimento formal de ligacao para `N_eff` (definicao de governanca):

$$
\bar\rho = \rho_{floor} + \sum_{j} k_j\,\beta_j, \quad k_j \ge 0
$$

$$
N_{eff} = \frac{N}{1+(N-1)\bar\rho}
$$

### 2.4 Parametros de confianca e aceite

- `epsilon_target = 1e-11`: erro nao detectado maximo admissivel.
- `A_target = 0.9999`: disponibilidade minima admissivel (99.99%).
- `alpha = 0.05`: nivel de confianca estatistica (95%).

Criterio global de aprovacao para G0..G4:

$$
UCB_{1-\alpha}(P_{UE}(T_*)) \le \epsilon_{target}
$$

$$
LCB_{1-\alpha}(1-P_{unavail}(T_*)) \ge A_{target}
$$

e sem risco critico aberto em chave, operacao ou causa comum.

## 3. Gate G0 - Formalizacao
Objetivo do gate: fechar `Theta`, threat model, budgets `epsilon_i`, parametros (`epsilon_target`, `A_target`, `alpha`) e criterios formais de Go/No-Go.

| ID da Prova | Metrica Alvo | Metodo de Verificacao | Evidencia Requerida |
|---|---|---|---|
| PO-1 (Codificacao e Sincronizacao) | `P(loss_sync) <= 1e-10`; `epsilon_ch + epsilon_sync` explicitamente alocados. | Model checking probabilistico do canal nao-IID (Markov/HMM) e prova de invariantes de sincronizacao. | Especificacao matematica versionada; certificados de invariantes; planilha assinada de budget `epsilon`. |
| PO-2 (Verificador Híbrido / Fail-stop) | `Pr(false_accept) <= epsilon_ver`; criterio fail-stop formalizado: `Pr(false_accept) << Pr(false_abort)`; arquitetura de duas camadas formalizada. | Especificacao formal do Verificador Híbrido (Lógica Molecular + ZKP In-Silico): camada fisica com marcadores bioquimicos (Toehold Switches e Barcodes Moleculares) para triagem fail-stop e camada computacional com ZKP criptografico apos sequenciamento. | Provas de safety/liveness por camada; matriz de confusao teorica; politica de fail-stop assinada; evidencia de validacao de integridade sem exposicao do segredo. |
| PO-3 (Consenso e Causa Comum) | Equacao `beta -> rho -> N_eff` aprovada; `N_eff` alvo e limites por cenario definidos. | Analise de confiabilidade, FTA e derivacao dos coeficientes `k_j` por componente de `beta`. | Arvore de falhas; relatorio de sensibilidade `beta_j -> rho`; baseline de `N_eff`. |
| PO-4 (Chaves e Governanca) | `Pr(key_compromise_and_accept_invalid) <= epsilon_key`; ciclo de vida completo definido. | Prova formal do protocolo threshold e analise de estados de chave (geracao, rotacao, revogacao, recuperacao). | Especificacao PKI/threshold; matriz de transicao; controles compensatorios aprovados. |
| PO-5 (Operacao e Disponibilidade) | `A_target` e `alpha` definidos; regra `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target` formalizada. | STPA/FMEA operacional e modelagem de indisponibilidade com trilha de observabilidade minima obrigatoria. | Mapa de telemetria; plano de resposta; criterio LCB de disponibilidade aprovado. |

## 4. Gate G1 - In-silico
Objetivo do gate: estimar `P_UE` e `P_unavail` com eventos raros, Importance Sampling e limites de confianca em ambiente computacional controlado.

| ID da Prova | Metrica Alvo | Metodo de Verificacao | Evidencia Requerida |
|---|---|---|---|
| PO-1 (Codificacao e Sincronizacao) | `UCB_{1-alpha}(P_decode_fail) <= 1e-11`; consumo de `epsilon_ch + epsilon_sync` dentro do budget. | Monte Carlo raro + Importance Sampling com injecao de burst errors e eventos estruturais correlacionados. | Curvas de convergencia; intervalos de confianca; relatorio de consumo de budget por dominio. |
| PO-2 (Verificador Híbrido / Fail-stop) | `UCB_{1-alpha}(P_false_accept) <= epsilon_ver`; fail-stop dominante em entradas invalidas; validacao por camada fisica e computacional. | Validacao da camada fisica via marcadores bioquimicos de integridade (Toehold Switches e Barcodes Moleculares) e da camada computacional via ZKP criptografico executado apos sequenciamento, com analise ROC/PR. | Matriz de confusao consolidada por camada; relatorio de dominancia fail-stop; logs de abort em entradas invalidas; evidencia de verificacao sem exposicao do segredo. |
| PO-3 (Consenso e Causa Comum) | `UCB_{1-alpha}(beta_shared) <= 1e-6`; `N_eff/N >= 0.8` via `rho` estimado. | Simulacao distribuida com falhas de causa comum (amostra, pipeline, modelo, chave, adversarial). | Tabela de correlacoes; decomposicao empirica de `beta_shared`; estimativa de `rho` e `N_eff`. |
| PO-4 (Chaves / Threshold / Recuperacao) | `P(key_compromise_and_accept_invalid) <= epsilon_key`; `P(recovery_success) >= 1 - epsilon_live`. | Simulacao de colusao, perda parcial de fragmentos e politicas de recuperacao. | Evidencias de intrusao logica; trilhas de recuperacao; taxa de sucesso por cenario. |
| PO-5 (Operacao e Disponibilidade) | `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target`; observabilidade critica `>= 99.99%`. | Digital twin fim-a-fim com chaos testing controlado e analise de indisponibilidade. | Dashboard de disponibilidade; relatorio de indisponibilidade; cobertura de alarmes e runbooks. |

## 5. Gate G2 - Heterogeneidade
Objetivo do gate: reduzir `beta_modelo` por validacao cruzada entre stacks independentes e provar robustez de decisao multi-implementacao.

| ID da Prova | Metrica Alvo | Metodo de Verificacao | Evidencia Requerida |
|---|---|---|---|
| PO-1 (Codificacao e Sincronizacao) | Divergencia inter-stack de decode `<= 1e-7`; concordancia de frame `>= 0.999999`. | Differential decoding entre stacks independentes. | Matriz de divergencia por stack; relatorio de inconsistencias de frame. |
| PO-2 (Verificador Híbrido / Fail-stop) | Desvio de FAR entre stacks `<= 1e-8`; fail-stop consistente entre implementacoes. | Reexecucao da bateria de casos invalidos em multiplos verificadores hibridos independentes. | Matriz de confusao cruzada por stack; diff de decisoes por caso. |
| PO-3 (Consenso e Causa Comum) | `UCB_{1-alpha}(beta_modelo) <= 2e-7`; `UCB_{1-alpha}(beta_shared) <= 1e-6`; `N_eff/N >= 0.8`. | Analise de causa comum residual apos heterogeneidade e reestimacao de `rho`. | Relatorio de sensibilidade de `beta_modelo`; impacto medido em `rho` e `N_eff`. |
| PO-4 (Chaves / Interoperabilidade) | Divergencia inter-stack de reconstrucao de chave `<= 1e-9`. | Testes de interoperabilidade criptografica entre bibliotecas independentes. | Vetores de teste assinados; relatorio de compatibilidade inter-stack. |
| PO-5 (Operacao e Disponibilidade) | `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target` com stacks heterogeneos em paralelo. | Execucao paralela e reconciliacao automatica de artefatos e metricas. | Hashes de artefatos por stack; relatorio de reconciliacao e disponibilidade agregada. |

## 6. Gate G3 - Fault Injection Adversarial
Objetivo do gate: validar resiliencia sob ataques coordenados e falhas massivas de causa comum mantendo Safety, Liveness e Fail-stop.

| ID da Prova | Metrica Alvo | Metodo de Verificacao | Evidencia Requerida |
|---|---|---|---|
| PO-1 (Codificacao e Sincronizacao) | `UCB_{1-alpha}(P_decode_fail_adversarial) <= 5e-10`; perda silenciosa de sync `<= 1e-12`. | Campanhas adversariais com burst errors sincronizados, corrupcao de indices e falhas estruturais densas. | Relatorio de campanha adversarial; curvas de degradacao sob estresse. |
| PO-2 (Verificador Híbrido / Fail-stop) | `UCB_{1-alpha}(P_false_accept_adversarial) <= epsilon_ver`; fail-stop dominante em carga maliciosa. | Adversarial fuzzing no verificador hibrido com estresse de fronteira de decisao. | Matriz de confusao adversarial; evidencias de bloqueio e abort seguro. |
| PO-3 (Consenso e Causa Comum) | `UCB_{1-alpha}(beta_adversarial) <= 5e-7`; `UCB_{1-alpha}(beta_shared) <= 1e-6`; `N_eff/N >= 0.7` em modo degradado. | Chaos testing coordenado com falhas simultaneas em amostra, pipeline e modelo. | Logs de convergencia sob ataque; decomposicao de causa comum adversarial. |
| PO-4 (Chaves / Threshold) | Com colusao ate `t-1`, `P(key_compromise_and_accept_invalid) <= epsilon_key`. | Simulacao de replay, colusao e injecao de fragmentos invalidos na reconstrucao. | Trilhas de tentativa de replay; evidencias de bloqueio de comprometimento. |
| PO-5 (Operacao e Disponibilidade) | `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target`; MTTD e MTTC dentro dos limites aprovados. | Exercicios adversariais operacionais (purple-team) com deteccao e contencao automatizada. | Cronologia de incidente; dashboards de disponibilidade; tempos de resposta medidos. |

## 7. Gate G4 - Longitudinal
Objetivo do gate: comprovar estabilidade temporal, controle de deriva de `Theta` e sustentabilidade de Safety/Liveness/Fail-stop em horizonte prolongado.

| ID da Prova | Metrica Alvo | Metodo de Verificacao | Evidencia Requerida |
|---|---|---|---|
| PO-1 (Codificacao e Sincronizacao) | `UCB_{1-alpha}(P_decode_fail(t)) <= 1e-10` em janelas sucessivas; deriva de sync `<= 1e-8` por janela. | Monitoramento longitudinal com cartas de controle e testes de tendencia. | Serie temporal de decode fail; relatorio de breakpoints e estabilidade. |
| PO-2 (Verificador Híbrido / Fail-stop) | Estabilidade de FAR por janela e manutencao da dominancia fail-stop no tempo. | Reexecucao periodica de suites de validacao e analise de drift do verificador hibrido. | Historico de matriz de confusao por janela; relatorio de drift e recalibracao. |
| PO-3 (Consenso e Causa Comum) | `UCB_{1-alpha}(beta_shared(t)) <= 1e-6`; deriva de `beta_modelo` controlada; `N_eff/N >= 0.8`. | Reestimacao periodica de `beta_j`, `rho` e `N_eff` com alarmes de deriva de `Theta`. | Serie temporal de `beta_shared`, `rho`, `N_eff`; evidencias de recalibracao. |
| PO-4 (Chaves e Governanca) | Em ciclos de rotacao, `P(key_compromise_and_accept_invalid) <= epsilon_key`; recuperacao sustentada. | Campanhas recorrentes de rotacao/revogacao/recuperacao com compatibilidade interversao. | Trilhas de ciclo de vida por periodo; resultados de recuperacao por janela. |
| PO-5 (Operacao e Disponibilidade) | `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target` sustentado longitudinalmente. | Auditoria temporal de disponibilidade e continuidade de cadeia de custodia. | Relatorio longitudinal de disponibilidade; mapa de lacunas de observabilidade. |

## 8. Regra de decisao por Gate

- Gate G0 aprovado se:
  - `Theta`, threat model e budgets `epsilon_i` estiverem fechados e versionados.
  - `epsilon_target`, `A_target` e `alpha` estiverem formalmente aprovados.
  - Todos os PO-1..PO-5 em status `PASS` de formalizacao.

- Gate G1 aprovado se:
  - Todos os PO-1..PO-5 estiverem `PASS` com intervalos `1-alpha` declarados.
  - `UCB_{1-alpha}(P_UE(T_*)) <= epsilon_target`.
  - `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target`.

- Gate G2 aprovado se:
  - Todos os PO-1..PO-5 estiverem `PASS` em validacao cruzada inter-stack.
  - `UCB_{1-alpha}(beta_modelo) <= 2e-7`.
  - `UCB_{1-alpha}(P_UE(T_*))` e `LCB_{1-alpha}(1-P_unavail(T_*))` mantidos no SLA.

- Gate G3 aprovado se:
  - Todos os PO-1..PO-5 estiverem `PASS` sob campanhas adversariais coordenadas.
  - `UCB_{1-alpha}(beta_shared) <= 1e-6` apos stress massivo.
  - Liveness e fail-stop preservados no envelope adversarial.

- Gate G4 aprovado se:
  - Todos os PO-1..PO-5 estiverem `PASS` no horizonte longitudinal definido.
  - Limites de deriva de `Theta`, `beta_shared`, `rho` e `N_eff` dentro do envelope.
  - Sem risco critico aberto sem plano de remediacao assinado.

## 9. Clausula formal de emissao de claim
E vedada a emissao de claim de confiabilidade de missao critica sem aprovacao simultanea de:

1. todos os PO-1..PO-5 em todos os gates aplicaveis (G0..G4),
2. `UCB_{1-alpha}(P_UE(T_*)) <= epsilon_target`,
3. `LCB_{1-alpha}(1-P_unavail(T_*)) >= A_target`,
4. criterio de fail-stop dominante,
5. ausencia de risco critico aberto em chave, operacao ou causa comum.

Claim final permitido:

"Determinismo de engenharia com horizonte epsilon, sob envelope Theta auditado e budgets de risco explicitamente verificados."

## 10. Checklist minimo de artefatos auditaveis

- Especificacao formal versionada (`Theta`, threat model, budgets `epsilon_i`).
- Relatorios de simulacao com seeds, versoes e hash de datasets sinteticos.
- Evidencias de consenso (logs de quorum, correlacao, falhas de modo comum).
- Evidencias inter-stack (differential decoding, matriz de divergencia, reconciliacao de artefatos).
- Evidencias adversariais (campanhas coordenadas, tempos de deteccao e contencao).
- Evidencias longitudinais (series temporais, drift de `Theta`, cartas de controle).
- Evidencias de safety e fail-stop (matriz de confusao, FAR/FRR, dominancia de abort seguro).
- Evidencias de liveness/availability (`LCB_{1-alpha}(1-P_unavail(T_*))`).
- Evidencias de governanca de chave (compromisso, recuperacao, revogacao).

## 11. Observacao de uso
Este manual e intencionalmente restrito a modelagem, validacao e auditoria de confiabilidade. Nao contem procedimentos biologicos experimentais nem instrucoes operacionais de bancada.

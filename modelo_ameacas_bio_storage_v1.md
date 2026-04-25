# Modelo de Ameacas e Envelope de Incerteza - Bio Storage v1

## 1. Escopo e objetivo
Este documento define o Envelope de Incerteza (`Theta`) do projeto Sanguine Ledger sob criterio estrito do Manual v8.0.

Objetivo:

- mapear vetores de falha que impactam Safety, Liveness e Fail-stop,
- decompor risco no formato `epsilon` obrigatorio,
- conectar causa comum (`beta_shared`) ao calculo de `N_eff`.

## 2. Envelope de Incerteza (Theta)

O espaco de incerteza operacional e modelado como:

$$
\Theta = \Theta_{bio} \cup \Theta_{infra} \cup \Theta_{logic} \cup \Theta_{ops}
$$

Com parametros de confianca explicitamente governados:

- `epsilon_target`: erro nao detectado maximo admissivel.
- `A_target`: disponibilidade minima admissivel.
- `alpha`: nivel de confianca estatistica.

## 3. Decomposicao de risco obrigatoria (v8.0)

### 3.1 Bound de Safety por dominio

$$
P_{UE}(T) \le
\epsilon_{ch}+\epsilon_{sync}+\epsilon_{ver}+\epsilon_{cons}+\epsilon_{key}+\epsilon_{ops}+\epsilon_{adv}
$$

Regra de fechamento:

$$
\sum_i \epsilon_i \le \epsilon_{target}
$$

### 3.2 Mapeamento Theta -> epsilon_i

| Dominio epsilon | Classe de risco primaria | Vetores dominantes em Theta | Saida de auditoria |
|---|---|---|---|
| `epsilon_ch` | Erro de canal | mutacao de ponto, vies de leitura, ruido biofisico de simbolo | bound de erro de simbolo/canal |
| `epsilon_sync` | Erro de sincronizacao | rearranjos, burst errors, corrupcao de delimitador/frame | bound de perda de frame e realinhamento |
| `epsilon_ver` | Erro de verificador | bypass logico, forca bruta no Verificador Híbrido (Lógica Molecular + ZKP In-Silico), drift de fronteira de decisao | `Pr(false_accept)` e dominancia fail-stop |
| `epsilon_cons` | Erro de consenso | correlacao inter-nos, quorum degradado, falha comum de decisao | bound de commit invalido por quorum |
| `epsilon_key` | Erro de governanca de chave | colusao threshold, replay, reconstrucao indevida | bound de comprometimento de chave |
| `epsilon_ops` | Erro operacional | cadeia de custodia, erro de processo, indisponibilidade operacional | bound de falha operacional e `P_unavail` |
| `epsilon_adv` | Erro adversarial | sabotagem coordenada, contaminacao maliciosa, stress de modo comum | bound residual sob ataque coordenado |

## 4. Matriz de ameacas com impacto em epsilon

## 4.1 Ameacas biologicas (Bio-Stochastic)

| ID | Vetor de ameaca | Impacto primario em epsilon | Impacto estimado (pre -> residual) | Mitigacao principal | POs conectadas |
|---|---|---|---|---|---|
| BIO-01 | Mutacao de ponto | `epsilon_ch` | `3e-13 -> 8e-14` | ECC com margem e monitoramento de taxa de erro | PO-1, PO-2 |
| BIO-02 | Rearranjos estruturais | `epsilon_sync` | `4e-13 -> 1e-13` | Invariantes de sincronizacao + campanha de burst correlacionado | PO-1 |
| BIO-03 | Deriva genetica | `epsilon_ch`, `epsilon_cons` | `2e-13 -> 7e-14` | Reestimacao periodica de parametros e recalibracao de envelope | PO-1, PO-3 |
| BIO-04 | Contaminacao por biofilmes externos | `epsilon_ops`, `epsilon_ver` | `2.5e-13 -> 6e-14` | Proveniencia assinada e regras de abort conservador | PO-5, PO-2 |
| BIO-05 | Estresse metabolico | `epsilon_ops` | `1.2e-13 -> 4e-14` | Limites operacionais e monitoramento de estabilidade | PO-5, PO-2 |

## 4.2 Ameacas de infraestrutura

| ID | Vetor de ameaca | Impacto primario em epsilon | Impacto estimado (pre -> residual) | Mitigacao principal | POs conectadas |
|---|---|---|---|---|---|
| INF-01 | Vies sistematico de leitura (basecalling) | `epsilon_ch`, `epsilon_cons` | `3.5e-13 -> 9e-14` | Stacks heterogeneos e validacao cruzada | PO-1, PO-3, PO-5 |
| INF-02 | Descalibracao de sensores | `epsilon_ops` | `1.8e-13 -> 5e-14` | Calibracao rastreada e intertravamento logico | PO-5 |
| INF-03 | Ruido de rede (60Hz) | `epsilon_ch` | `1.4e-13 -> 4e-14` | Filtragem e robustez em digital twin | PO-1, PO-5 |
| INF-04 | Falha termica | `epsilon_ops`, `epsilon_ver` | `2.2e-13 -> 6e-14` | Telemetria de trip e fail-safe para abort | PO-5, PO-2 |

## 4.3 Ameacas logicas e criptograficas

| ID | Vetor de ameaca | Impacto primario em epsilon | Impacto estimado (pre -> residual) | Mitigacao principal | POs conectadas |
|---|---|---|---|---|---|
| LOG-01 | Colisoes de hash em fragmentos curtos | `epsilon_sync`, `epsilon_key` | `2.8e-13 -> 7e-14` | Dominio de hash efetivo ampliado + namespaces | PO-1, PO-4 |
| LOG-02 | Falha de sincronizacao de frame | `epsilon_sync`, `epsilon_ver` | `4.0e-13 -> 1e-13` | Invariantes formais de frame e abort deterministico | PO-1, PO-2 |
| LOG-03 | Comprometimento de nos SSS | `epsilon_key` | `3.2e-13 -> 8e-14` | Threshold robusto e segregacao de papeis | PO-4, PO-5 |
| LOG-04 | Forca bruta no verificador in-silico via sequenciamento corrompido | `epsilon_ver`, `epsilon_adv` | `2.0e-13 -> 5e-14` | Rate limiting, validacao de integridade de sequenciamento e stress adversarial de fronteira no Verificador Híbrido (Lógica Molecular + ZKP In-Silico) | PO-2, PO-4 |

## 4.4 Ameacas adversariais e operacionais

| ID | Vetor de ameaca | Impacto primario em epsilon | Impacto estimado (pre -> residual) | Mitigacao principal | POs conectadas |
|---|---|---|---|---|---|
| OPS-01 | Sabotagem fisica | `epsilon_adv`, `epsilon_ops` | `3.8e-13 -> 9e-14` | Defesa fisica em camadas + sensores anti-violacao | PO-5, PO-3 |
| OPS-02 | Erro de manipulacao | `epsilon_ops` | `2.6e-13 -> 7e-14` | Controle duplo e observabilidade de processo | PO-5 |
| OPS-03 | Degradacao da cadeia de custodia | `epsilon_ops`, `epsilon_key` | `3.0e-13 -> 8e-14` | Assinatura ponta a ponta e auditoria cruzada | PO-5, PO-4 |

## 5. Causa comum: procedimento formal beta -> rho -> Neff

### 5.1 Decomposicao

$$
\beta_{shared} = \beta_{substrato}+\beta_{amostra}+\beta_{pipeline}+\beta_{modelo}+\beta_{chave}+\beta_{adversarial}
$$

### 5.2 Conversao para correlacao residual

Para cada janela de validacao `w`, adotar:

$$
\bar\rho_w = \rho_{floor} + \sum_j k_j\,\beta_{j,w}
$$

com `k_j` estimados em G0 e recalibrados em G4 quando houver drift de `Theta`.

### 5.3 Impacto em numero efetivo de nos

$$
N_{eff,w} = \frac{N_w}{1+(N_w-1)\bar\rho_w}
$$

Criterio operacional:

- `N_eff/N >= 0.8` no regime nominal.
- modo degradado permitido apenas em G3, com limite explicitamente aprovado.

### 5.4 Evidencia obrigatoria por gate

- G1: estimativa inicial de `beta_j`, `rho` e `N_eff`.
- G2: reducao de `beta_modelo` por heterogeneidade e efeito mensuravel em `rho`.
- G3: robustez de `beta_shared` e `N_eff` sob ataque coordenado.
- G4: estabilidade temporal de `beta_j -> rho -> N_eff` com controle de drift.

## 6. Alinhamento de Gates (G0..G4)

- G0: fechamento formal de `Theta`, budget `epsilon_i`, `alpha`, `A_target`.
- G1: estimacao estatistica de `P_UE` e `P_unavail` com confianca `1-alpha`.
- G2: validacao cruzada inter-stack para reduzir `beta_modelo`.
- G3: fault injection adversarial e reestimacao de risco sob causa comum massiva.
- G4: validacao longitudinal e recalibracao controlada quando houver drift.

## 7. Criterios de aceitacao do modelo de ameacas

O documento e considerado conforme v8.0 quando:

- todas as ameacas estiverem mapeadas para ao menos um `epsilon_i` e uma PO;
- o fechamento `sum(epsilon_i) <= epsilon_target` estiver demonstrado;
- o procedimento `beta -> rho -> N_eff` estiver aplicado com evidencia por gate;
- os criterios de confianca forem expressos com `alpha`, `UCB_{1-alpha}` e `LCB_{1-alpha}`;
- Safety, Liveness e Fail-stop forem auditados em G0..G4.

## 8. Clausula de claim
Sem aprovacao de todas as POs e sem cumprimento dos limites de confianca (`UCB_{1-alpha}(P_UE)`, `LCB_{1-alpha}(1-P_unavail)`), nao ha claim permitido de confiabilidade de missao critica.

Claim permitido apos conformidade total:

"Determinismo de engenharia com horizonte epsilon, sob envelope Theta auditado e budgets de risco explicitamente verificados."

## 9. Nota de governanca
Este documento estabelece modelagem de risco e governanca de confiabilidade. Nao inclui protocolos experimentais, procedimentos de bancada ou parametros biologicos operacionais.

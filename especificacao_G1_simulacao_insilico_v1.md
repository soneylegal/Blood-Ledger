# Especificacao G1 - Simulacao In-silico v1

## 1. Objetivo e escopo
Este documento define o Protocolo de Simulacao e Validacao Computacional para o Gate G1 do projeto Sanguine Ledger.

Objetivo primario:

- Demonstrar quantitativamente as Obrigacoes de Prova PO-1 a PO-5.
- Estimar limites superiores de risco de safety em regime de evento raro.
- Estimar indisponibilidade para validacao formal de liveness.
- Verificar dissipacao de causa comum via metricas de correlacao e robustez de consenso.

Metas de referencia (alinhadas ao Manual v8.0):

- $UCB_{1-\alpha}(P_{UE}(T^*)) \le \epsilon_{target}$
- $LCB_{1-\alpha}(1-P_{unavail}(T^*)) \ge A_{target}$
- $N_{eff} \ge 0.8N$
- $UCB_{1-\alpha}(\beta_{shared}) \le 10^{-6}$

Parametros obrigatorios de confianca:

- `alpha`: nivel de confianca estatistica.
- `epsilon_target`: erro nao detectado maximo admissivel.
- `A_target`: disponibilidade minima admissivel.

## 2. Arquitetura do motor de simulacao (Digital Twin)

## 2.1 Visao em camadas
O motor e modelado como um Digital Twin distribuido com cinco camadas acopladas:

1. Camada de canal biologico (nao-IID): mutacao de ponto, burst errors, rearranjos e deriva.
2. Camada de infraestrutura: ruido de instrumentacao, drift termico e interferencia eletrica (60Hz).
3. Camada logica/criptografica: sincronizacao de frame, colisao de hash, Verificador Híbrido (Lógica Molecular + ZKP In-Silico) e threshold secret sharing.
4. Camada de consenso: correlacao entre nos, quorum e commit/abort.
5. Camada operacional: proveniencia, cadeia de custodia e perturbacoes humanas/adversariais.

## 2.2 Modelo de estado estocastico
O estado do sistema no passo $t$ e representado por:

$$
X_t = (B_t, I_t, L_t, C_t, O_t)
$$

onde:

- $B_t$: estado biologico do payload/canal
- $I_t$: estado de infraestrutura/sensoriamento
- $L_t$: estado logico/criptografico
- $C_t$: estado de consenso distribuido
- $O_t$: estado operacional/proveniencia

A dinamica segue cadeia de Markov de ordem 1:

$$
P(X_{t+1}\mid X_t, u_t) = P_B \cdot P_I \cdot P_L \cdot P_C \cdot P_O
$$

com parametros calibrados por envelope $\Theta$ e cenarios de estresse.

## 2.3 Submodelo de ruido eletrico e drift
A leitura sintetica incorpora componente periodica e componente estocastica:

$$
n_t = \phi n_{t-1} + A\sin\left(2\pi f_0 t\Delta t + \varphi\right) + \eta_t,
\quad f_0 = 60\,Hz,
\quad \eta_t \sim \mathcal{N}(0,\sigma^2)
$$

Drift termico e modelado por processo lento:

$$
T_{t+1} = T_t + \kappa(\bar{T}-T_t) + \xi_t
$$

com impacto no erro de leitura via funcao de transferencia $g(T_t)$.

## 2.4 Artefatos gerados pelo motor
Cada execucao produz:

- trilhas de estados por camada
- logs de falha/commit/abort
- pesos de Importance Sampling
- estatisticas de consenso e correlacao
- metricas por PO-1..PO-5

## 2.5 Verificador Híbrido (Lógica Molecular + ZKP In-Silico)
Para PO-2, a verificacao fail-stop e modelada em duas camadas acopladas:

- Camada fisica: marcadores bioquimicos de integridade (ex.: Toehold Switches e Barcodes Moleculares) acionam triagem fail-stop inicial.
- Camada computacional: apos o sequenciamento, um ZKP criptografico valida integridade sem exposicao do segredo.

O aceite de PO-2 exige dominancia fail-stop em ambas as camadas e consistencia entre sinais fisicos e verificacao in-silico.

## 3. Metodologia de eventos raros (Importance Sampling)

## 3.1 Problema
Estimativas diretas de $p \approx 10^{-12}$ por Monte Carlo ingenuo exigem custo impraticavel.
No estimador Bernoulli padrao:

$$
\hat{p}_{MC} = \frac{1}{N}\sum_{i=1}^{N} \mathbf{1}_{A}(X_i),
\quad
\mathrm{Var}(\hat{p}_{MC}) = \frac{p(1-p)}{N}
$$

para eventos muito raros, a convergencia em precisao relativa torna-se proibitiva.

## 3.2 Estimador com distribuicao de proposta
Amostragem e feita sob distribuicao de proposta $g(x)$, com pesos:

$$
w(x)=\frac{f(x)}{g(x)}
$$

Estimador nao-viesado:

$$
\hat{p}_{IS} = \frac{1}{N}\sum_{i=1}^{N} \mathbf{1}_{A}(X_i)w(X_i),
\quad X_i \sim g
$$

Variancia:

$$
\mathrm{Var}_g(\hat{p}_{IS})
= \frac{1}{N}\left(\mathbb{E}_g[\mathbf{1}_{A}(X)w(X)^2]-p^2\right)
$$

Objetivo de engenharia: escolher $g$ para concentrar massa de probabilidade em trajetorias proximas do conjunto de falha $A$ sem inflar excessivamente os pesos.

## 3.3 Propostas adaptativas
Sao usadas propostas por classe de risco:

- Proposal-BIO: aumenta probabilidade de burst/rearranjo.
- Proposal-INFRA: aumenta ruido 60Hz, drift termico e vies de leitura.
- Proposal-LOGIC: aumenta cenarios de colisao/sync fault/ataque ao Verificador Híbrido (Lógica Molecular + ZKP In-Silico).
- Proposal-OPS: aumenta falhas de proveniencia e sabotagem de pipeline.

A adaptacao e guiada por minimizacao de variancia empirica ponderada por PO.

## 3.4 Ganho de eficiencia
Ganho de eficiencia do Importance Sampling:

$$
G = \frac{\mathrm{Var}(\hat{p}_{MC})}{\mathrm{Var}(\hat{p}_{IS})}
$$

e tamanho efetivo de amostra ponderada:

$$
ESS = \frac{\left(\sum_{i=1}^{N} w_i\right)^2}{\sum_{i=1}^{N} w_i^2}
$$

Criterio operacional minimo recomendado:

- $G \ge 10^4$ para campanhas de alvo $10^{-12}$
- $ESS/N \ge 0.2$ para evitar degeneracao severa de pesos

## 4. Protocolos de injecao de falhas (Fault Injection)

## 4.1 Estrutura de campanha
Cada campanha possui:

- hipotese de falha
- operador de injecao
- varredura parametrica
- mapeamento para PO(s)
- criterio quantitativo de aprovacao

## 4.2 Campanhas por ameaca critica de Theta

| Campanha | Ameaca alvo | Operador de injecao | Parametros de varredura | POs alvo | Evidencia esperada |
|---|---|---|---|---|---|
| FI-01 | Burst errors e rearranjos (BIO-02) | Insercao de blocos de erro correlacionado no canal | comprimento de burst, taxa de transicao Markov, densidade de eventos | PO-1 | Curva de perda de frame, $UCB_{1-\alpha}(P_{decode\_fail})$, logs de recuperacao |
| FI-02 | Mutacao de ponto e deriva (BIO-01/BIO-03) | Perturbacao de taxa de mutacao por estado | grade de $\mu$, matriz de transicao e janela temporal | PO-1, PO-3 | Sensibilidade de reconstruibilidade e deslocamento de risco residual |
| FI-03 | Vies de basecalling (INF-01) | Distorcao sistematica no modelo de leitura | bias por classe de base, SNR, offsets por lote | PO-1, PO-5 | Divergencia inter-pipeline e taxa de erro correlacionado |
| FI-04 | Ruido 60Hz e drift termico (INF-03/INF-04) | Injecao de componente senoidal e drift em sensores | amplitude, fase, $\kappa$, tempo de exposicao | PO-5, PO-2 | Taxa de abort, estabilidade de decisao e alarmes de trip |
| FI-05 | Colisoes de hash (LOG-01) | Geracao forcada de fragmentos adversariais curtos | tamanho de fragmento, espaco de hash efetivo | PO-1, PO-4 | Probabilidade de commit ambiguo e taxa de deteccao do verificador |
| FI-06 | Falha de frame sync (LOG-02) | Perturbacao de delimitadores e indices | densidade de corrupcao de indice, offset aleatorio | PO-1, PO-2 | Frequencia de abort deterministico e falso aceite residual |
| FI-07 | Comprometimento de nos SSS (LOG-03) | Modelo de corrupcao por subconjunto de nos | cardinalidade de nos comprometidos, limiar $t$ | PO-4, PO-5 | $P(key\_compromise\_and\_accept\_invalid)$ e tempos de recuperacao |
| FI-08 | Forca bruta no verificador in-silico via sequenciamento corrompido (LOG-04) | Geracao de consultas adversariais em lote com sequenciamento intencionalmente corrompido | taxa de tentativa, diversidade de payload invalido, taxa de corrupcao de leitura | PO-2, PO-4 | FAR/FRR, curva ROC/PR e comportamento fail-stop |
| FI-09 | Sabotagem/custodia (OPS-01/OPS-03) | Quebra de proveniencia e adulteracao de metadados | ponto de ataque no pipeline, atraso de deteccao | PO-5, PO-3 | Cobertura de deteccao, trilha de auditoria e impacto em $\beta_{shared}$ |

## 4.3 Controle de causa comum durante fault injection
Para cada campanha, medir:

- correlacao de falha entre nos ($\rho_{ij}$)
- contribuicao marginal em $\beta_{shared}$ por componente
- degradacao de $N_{eff}/N$

Procedimento formal de ligacao causa comum -> correlacao -> quorum:

$$
\bar\rho = \rho_{floor} + \sum_j k_j\,\beta_j
$$

$$
N_{eff}=\frac{N}{1+(N-1)\bar\rho}
$$

Meta de aceitacao transversal:

- $UCB_{1-\alpha}(\beta_{shared}) \le 10^{-6}$
- $N_{eff}/N \ge 0.8$

## 5. Principios de validade experimental

- Reprodutibilidade: cenarios, parametros e sementes devem ser versionados.
- Auditabilidade: cada campanha deve produzir trilha verificavel de metricas e decisoes.
- Comparabilidade: resultados devem ser avaliados com os mesmos criterios de confianca e mesma definicao de evento.
- Rastreabilidade: cada evidencia deve ser vinculada a uma PO e a um gate.

## 6. Metricas de sucesso e criterios de parada

## 6.1 Metricas primarias por PO

- PO-1: $UCB_{1-\alpha}(P_{decode\_fail})$, taxa de recuperacao de frame e consumo de budget $\epsilon_{ch}+\epsilon_{sync}$
- PO-2: $UCB_{1-\alpha}(FAR)$, $UCB_{1-\alpha}(P_{UE\_local})$, taxa fail-stop em entradas invalidas na camada fisica (Toehold Switches e Barcodes Moleculares) e na camada computacional (ZKP apos sequenciamento)
- PO-3: $UCB_{1-\alpha}(\beta_{shared})$, $N_{eff}/N$, probabilidade de commit invalido por quorum
- PO-4: $P(key\_compromise\_and\_accept\_invalid)$, taxa de recuperacao de segredo
- PO-5: $UCB_{1-\alpha}(P_{UE\_end\_to\_end})$, $LCB_{1-\alpha}(1-P_{unavail})$, cobertura de deteccao operacional

## 6.2 Limites de confianca (UCB/LCB)
Para estimadores ponderados, usar:

$$
UCB_{1-\alpha}(\hat{p}) = \hat{p} + z_{1-\alpha}\sqrt{\widehat{\mathrm{Var}}(\hat{p})}
$$

Para disponibilidade:

$$
LCB_{1-\alpha}(1-\widehat{P_{unavail}}) = 1-\widehat{P_{unavail}} - z_{1-\alpha}\sqrt{\widehat{\mathrm{Var}}(\widehat{P_{unavail}})}
$$

Para casos com contagem extremamente baixa, aplicar metodo conservador (ex. limite exato de Bernoulli para falha rara) antes do sign-off.

## 6.3 Convergencia e parada de campanha
A campanha encerra quando todas as condicoes forem satisfeitas:

1. Precisao relativa do estimador de risco:
$$
\frac{z_{1-\alpha}\sqrt{\widehat{\mathrm{Var}}(\hat{p})}}{\hat{p}} \le r_{target}
$$
Valor nominal congelado em G0: $r_{target} = 0.05$ (erro de estimacao maximo de 5%), uniforme para todas as POs. Recalibracao permitida por PO em G4 se justificada por evidencia de convergencia.

2. Estabilidade temporal do estimador (janela movel) abaixo de limiar $\delta$.

3. ESS minimo mantido:
$$
ESS/N \ge ESS_{min}
$$
Valor nominal congelado em G0: $ESS_{min} = 0.2$ (20% do $N$ total, limiar para evitar degeneracao severa dos pesos de Importance Sampling).

4. Todas as metas de gate satisfeitas simultaneamente:

- $UCB_{1-\alpha}(P_{UE}(T^*)) \le \epsilon_{target}$
- $LCB_{1-\alpha}(1-P_{unavail}(T^*)) \ge A_{target}$
- $UCB_{1-\alpha}(\beta_{shared}) \le 10^{-6}$
- $N_{eff} \ge 0.8N$

## 7. Entregaveis para auditoria G1

- Relatorio consolidado por PO-1..PO-5
- Curvas de convergencia e variancia por campanha
- Matriz de correlacao entre nos e decomposicao de $\beta_{shared}$
- Relatorio de disponibilidade com `LCB_{1-alpha}(1-P_unavail)`
- Evidencias de fault injection por ameaca critica de $\Theta$
- Manifesto de reproducibilidade (seeds, versoes, hashes)

## 8. Clausula de alinhamento de claim
Este protocolo contribui para o claim final apenas se os limites de Safety, Liveness e Fail-stop forem aprovados com confianca `1-alpha` no contexto de G0..G4.

## 9. Limites de uso
Este documento cobre exclusivamente simulacao computacional, modelagem estatistica e validacao de confiabilidade. Nao inclui procedimentos biologicos experimentais nem instrucoes de manipulacao de bancada.
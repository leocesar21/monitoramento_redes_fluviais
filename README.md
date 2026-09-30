# Monitoramento Inteligente de Águas Pluviais via Análise Topológica e Dinâmica de Redes Complexas

Código do Projeto Final de Curso de Leonardo César de Oliveira
(Engenharia de Controle e Automação, UFMG, 2026).
Orientador: Prof. Dr. Leandro Freitas de Abreu.

Repositório: <https://github.com/leocesar21/monitoramento_redes_fluviais>

O código reproduz **todos os resultados numéricos, tabelas e figuras** dos Capítulos 3 e 4.
A tag `v1.1` corresponde ao texto final do PFC.

## Como reproduzir

```bash
git clone https://github.com/leocesar21/monitoramento_redes_fluviais.git
cd monitoramento_redes_fluviais
git checkout v1.1
python -m venv venv
source venv/bin/activate        # no Windows: venv\Scripts\activate
pip install -r requirements.txt
python executar_tudo.py
```

A execução completa leva cerca de 1 minuto. Ao final, `resultados/valores_do_pfc.md` lista cada número citado no texto junto com a seção de origem. Rodar de novo produz arquivos idênticos byte a byte.

> Se a instalação do `pyswmm` falhar ao compilar a dependência `julian`, instale-a antes com
> `pip install julian --no-build-isolation` e depois repita `pip install -r requirements.txt`.

### Ambiente da execução de referência

| Componente | Versão |
|---|---|
| Python | 3.11.15 |
| NumPy | 2.4.4 |
| SciPy | 1.17.1 |
| pandas | 3.0.2 |
| matplotlib | 3.10.9 |
| NetworkX | 3.6.1 |
| PySWMM | 2.1.0 |
| swmm-toolkit | 0.17.0 |
| Motor SWMM | 5.2.4 |

## Sequência de execução (`executar_tudo.py`)

1. `capitulo3/exemplo_sintetico.py`: exemplo de 5 nós (Cap. 3).
2. `capitulo4/gerar_inp.py`: escreve os `.inp` a partir de `rede.py` e `chuva.py`.
3. `capitulo4/simular_swmm.py`: simulações de referência no SWMM.
4. `capitulo4/calibrar.py`: calibração de α e p, só com o evento A.
5. `capitulo4/observabilidade.py`: monta o modelo calibrado e faz a análise topológica, a observabilidade e a busca gulosa.
6. `capitulo4/figura_topologia.py`: gera a Figura 4.1.
7. `capitulo4/estimacao.py`: Filtro de Kalman nos três eventos e métricas.
8. `capitulo4/robustez.py`: configurações de sensores e níveis de ruído.
9. Relatório `resultados/valores_do_pfc.md`.

## Estrutura

| Arquivo | O que faz | Onde aparece no PFC |
|---|---|---|
| `capitulo3/exemplo_sintetico.py` | Rede de 5 nós: A, L, C, Ad e Bd; observabilidade; κ(O); Filtro de Kalman; RMSE; ganho estacionário | Cap. 3, Tabela 3.2, Figura 3.2 |
| `capitulo4/rede.py` | Topologia, geometria dos condutos e parâmetros das sub-bacias | Seção 4.1 |
| `capitulo4/chuva.py` | Hietogramas dos eventos A, B e C | Tabela 4.3 |
| `capitulo4/gerar_inp.py` | Escreve os arquivos `.inp` do SWMM | Seção 4.1 |
| `capitulo4/simular_swmm.py` | Roda o SWMM e salva o nível de todos os nós a cada 1 min, além dos erros de continuidade | Seções 4.1 e 4.5 |
| `capitulo4/modelo.py` | Lê a topologia do `.inp`, monta f_ij, L, C e C_h, discretiza, faz as verificações e implementa observabilidade, Filtro de Kalman e métricas | Seções 4.2, 4.4 e 4.6; Apêndice A |
| `capitulo4/calibrar.py` | Calibra α e p | Seção 4.2.1 |
| `capitulo4/observabilidade.py` | Estabilidade, centralidade, gramiano, κ(O), busca gulosa e teste PBH | Seções 4.2–4.4, Tabelas 4.1 e 4.2 |
| `capitulo4/figura_topologia.py` | Figura da rede com sensores | Figura 4.1 |
| `capitulo4/estimacao.py` | Filtro nos três eventos, erro de pico, defasagem, RMSE por grupo e diagnóstico do evento B | Seções 4.6 e 4.7, Tabela 4.4, Figura 4.2 |
| `capitulo4/robustez.py` | Configurações de sensores e níveis de ruído | Seção 4.8, Tabelas 4.5 e 4.6 |

## Origem das entradas do estimador

- **Do `.inp`:** topologia, cotas e comprimento, rugosidade e diâmetro de cada conduto.
- **De `chuva.py`:** os hietogramas. São os mesmos escritos no `.inp`.
- **De `rede.py`:** a fração impermeável das sub-bacias, usada na entrada do modelo.
- **De `modelo.py`:** Ts = 1 min, o fator 0,6, a perda de 0,03 min⁻¹ nas cabeceiras e D_ref = 0,40 m.
- **De `calibracao.json`:** α e p, gerados por `calibrar.py`.
- **De `observabilidade.json`:** a configuração de sensores, gerada pela busca gulosa.

Editar um `.inp` à mão **não** atualiza `chuva.py` nem `rede.py`. Para mudar a rede ou a chuva, edite esses arquivos e rode `executar_tudo.py`.

## Métricas

- **RMSE (raiz do erro quadrático médio) de cada nó:** RMSE_i = √(média_k e²_ik), com e_ik = ĥ_ik − h^SWMM_ik.
- **Valor das tabelas:** média aritmética dos RMSE nodais, (1/N) Σ_i RMSE_i. É diferente do RMSE agregado √(média_{i,k} e²_ik).
- **Diagnóstico do evento B:** F é o conjunto dos instantes em que as cabeceiras estão no valor convencional de 3,0 m. São reportados η_F (fração do erro quadrático total contida em F) e a média dos RMSE nodais fora de F. O filtro roda sobre a série completa; só a avaliação exclui F.

## Escolhas de implementação

- **Tempo, Cap. 4:** a referência é amostrada em t = 0, 1, …, 239 min. A chuva do instante k vale em [k, k+1) min, que é a retenção de ordem zero.
- **Filtro, Cap. 4:** x̂₀ = 0 (rede vazia) e P₀ = 0,01·I. O ganho é calculado por sistema linear, e a covariância é atualizada na forma de Joseph. O filtro é linear e não se impõe nível ≥ 0.
- **Ruído, Cap. 4:** cada experimento usa `numpy.random.default_rng(42)`. Os ruídos não são pareados entre configurações com conjuntos de sensores diferentes, e há uma única realização por cenário.
- **Tabela 4.6:** R = σ²·I em cada nível de ruído.
- **Busca gulosa:** em caso de empate, escolhe-se o primeiro nó na ordem J1, …, J15, O1.
- **Gramiano e κ(O):** horizonte de N = 16 blocos, estados em nível (m), matriz C_h.
- **Verificações:** matriz Metzler, balanço de volume em C (1ᵀC ≤ 0) e em C_h (1ᵀS C_h = 1ᵀC S), estabilidade, grafo acíclico e caminho até o exutório.
- **Capítulo 3:** usa `numpy.random.seed(42)`, com w_k e v_k sorteados nessa ordem, e P₀ = I₅.
  - As séries começam em q₀ = q̂₀ = 0. No passo k, u_k leva q_k a q_{k+1}, e o filtro usa y_{k+1}. O RMSE é calculado sobre k = 1, …, 120.
  - A convergência de P é o primeiro k com ‖P_k − P_120‖_F / ‖P_120‖_F < 10 %.
  - O "estado real" é limitado a valores ≥ 0; o filtro não é.

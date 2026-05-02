# 🚗 Sistema Fuzzy ACC — Controle de Cruzeiro Adaptativo

> Sistema de Controle de Cruzeiro Adaptativo (ACC) com Lógica Fuzzy em Python. Utiliza três entradas — distância, erro de velocidade e velocidade relativa — e 10 regras fuzzy para decidir entre frear, manter ou acelerar, seguindo o fluxo clássico de Mamdani.

---

## 📋 Índice

- [Instalação](#instalação)
- [Como executar](#como-executar)
- [Variáveis do sistema](#variáveis-do-sistema)
- [Base de regras](#base-de-regras)
- [Fluxo do sistema](#fluxo-do-sistema)
- [Exemplo de uso](#exemplo-de-uso)
- [Superfícies de controle](#superfícies-de-controle)

---

## ⚙️ Instalação

```bash
pip install numpy scikit-fuzzy matplotlib
```

---

## ▶️ Como executar

```bash
jupyter notebook sistema_fuzzy.ipynb
```

---

## 🔢 Variáveis do sistema

### Entradas

| Variável | Universo | Conjuntos | Funções |
|---|---|---|---|
| **Distância** | 0 a 100 m | Perto, Média, Longe | Trapezoidal, Triangular, Trapezoidal |
| **Erro de Velocidade** `v_alvo − v_atual` | −60 a 60 km/h | Negativo, Zero, Positivo | Trapezoidal, Triangular, Trapezoidal |
| **Velocidade Relativa** `v_frente − v_atual` | −100 a 100 km/h | Aproximando, Acompanhando, Afastando | Trapezoidal, Triangular, Trapezoidal |

### Saída

| Variável | Universo | Conjuntos | Funções |
|---|---|---|---|
| **Ação** | −100 a 100% | Frear, Manter, Acelerar | Triangular |

> **Ação negativa** = frear &nbsp;|&nbsp; **Ação zero** = manter &nbsp;|&nbsp; **Ação positiva** = acelerar

---

## 📏 Base de regras

As 10 regras estão organizadas em **3 níveis de prioridade**. O operador `E` aplica o **mínimo** entre os graus de pertinência (lógica de Mamdani).

| # | Distância | Vel. Relativa | Erro de Vel. | Ação | Prioridade |
|---|---|---|---|---|---|
| R1 | Perto | Aproximando | — | **Frear** | 🔴 Alta |
| R2 | Perto | Acompanhando | — | **Frear** | 🔴 Alta |
| R3 | Perto | Afastando | — | **Manter** | 🔴 Alta |
| R4 | Média | Aproximando | — | **Frear** | 🟡 Média |
| R5 | Média | Acompanhando | — | **Manter** | 🟡 Média |
| R6 | Média | Afastando | Positivo | **Acelerar** | 🟡 Média |
| R7 | Média | Afastando | Negativo | **Frear** | 🟡 Média |
| R8 | Longe | — | Positivo | **Acelerar** | 🟢 Baixa |
| R9 | Longe | — | Zero | **Manter** | 🟢 Baixa |
| R10 | Longe | — | Negativo | **Frear** | 🟢 Baixa |

---

## 🔄 Fluxo do sistema

```
Sensores → Fuzzificação → Avaliação das Regras → Implicação → Agregação → Defuzzificação → Ação
```

| Etapa | Descrição |
|---|---|
| **Fuzzificação** | Converte os valores numéricos dos sensores em graus de pertinência |
| **Avaliação das regras** | Aplica o operador AND (mínimo) entre as condições de cada regra |
| **Implicação** | Corta a função de pertinência da saída usando a força da regra |
| **Agregação** | Une todas as figuras cortadas em uma única figura (máximo) |
| **Defuzzificação** | Calcula o centróide da figura agregada e retorna um valor exato |

---

## 💡 Exemplo de uso

```python
velocidade_alvo   = 100  # km/h
velocidade_atual  = 95   # km/h
velocidade_frente = 80   # km/h
dist_val          = 80   # m

# Cálculo automático das entradas derivadas
erro_val         = velocidade_alvo - velocidade_atual    # = +5 km/h
vel_relativa_val = velocidade_frente - velocidade_atual  # = -15 km/h

simulador.input['distancia']           = dist_val
simulador.input['erro_velocidade']     = erro_val
simulador.input['velocidade_relativa'] = vel_relativa_val

simulador.compute()
print(simulador.output['acao'])  # → 44.68% do pedal (acelerar levemente)
```

**Interpretação:** O carro está a 80 m do veículo da frente (longe), 5 km/h abaixo da meta, mas a velocidade relativa de −15 km/h indica que o carro da frente está freando. O sistema decide **acelerar levemente** (44,68%), equilibrando a meta de velocidade com a atenção ao risco.

---

## 📊 Superfícies de controle

Como o sistema possui 3 entradas, cada gráfico fixa uma variável e exibe a relação entre as outras duas e a saída. Isso gera 3 superfícies complementares.

### Gráfico 1 — Distância × Erro de Velocidade
> *Velocidade Relativa fixada em 0 km/h — trânsito estável*

- **Distância pequena + Erro positivo:** mesmo abaixo da meta, o sistema **freia** — proximidade tem prioridade.
- **Distância grande + Erro positivo:** via livre e abaixo da meta → sistema **acelera**.
- **Transição suave:** em situações intermediárias, o sistema pondera os fatores gradualmente.

### Gráfico 2 — Distância × Velocidade Relativa
> *Erro de Velocidade fixado em +5 km/h — levemente abaixo da meta*

- **Velocidade Relativa negativa:** o sistema **freia**, independente da distância atual.
- **Distância grande + Velocidade Relativa positiva:** situação segura → sistema **acelera**.
- **Queda abrupta:** quando a velocidade relativa fica muito negativa, a reação é intensa mesmo com distância razoável.

### Gráfico 3 — Erro de Velocidade × Velocidade Relativa
> *Distância fixada em 60 m — distância média*

- **Vel. Relativa negativa + Erro negativo:** duplo risco → **freada máxima**.
- **Vel. Relativa positiva + Erro positivo:** carro se afasta e há espaço → **aceleração máxima**.
- **Região central:** zona de equilíbrio onde o sistema tende a **manter a velocidade**.

### 🏆 Hierarquia de prioridades

A análise conjunta dos três gráficos revela a hierarquia implementada na base de regras:

```
Segurança (Vel. Relativa) > Distância > Meta de Velocidade (Erro)
```

### 🗺️ Legenda das cores

| Cor | Significado |
|---|---|
| 🟥 Vermelho | Frear (Ação ≈ −100%) |
| 🟨 Amarelo | Manter velocidade (Ação ≈ 0%) |
| 🟩 Verde | Acelerar (Ação ≈ +100%) |

---

## 🛠️ Tecnologias

- [Python 3](https://www.python.org/)
- [NumPy](https://numpy.org/)
- [scikit-fuzzy](https://pythonhosted.org/scikit-fuzzy/)
- [Matplotlib](https://matplotlib.org/)

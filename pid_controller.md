# O Controlador PID no Sistema de Controle Longitudinal

## 1. O que é um Controlador PID

PID significa **Proporcional-Integral-Derivativo**. É um dos algoritmos de 
controle mais usados na engenharia, presente em praticamente qualquer sistema 
que precise fazer uma grandeza física **atingir e se manter em um valor 
desejado** — seja a temperatura de um forno, a altitude de um drone, ou, no 
nosso caso, a velocidade de um carro.

### 1.1 A ideia central: controle em malha fechada

Um controlador PID funciona dentro de uma **malha fechada** (*closed loop*): 
o sistema mede constantemente a diferença entre o que se deseja (**referência** 
ou *setpoint*) e o que está realmente acontecendo (**valor medido**). Essa 
diferença é chamada de **erro**:

```
erro = referência − valor_medido
```

A cada instante, o PID observa esse erro e calcula uma **ação de controle** — 
um comando que tenta reduzir esse erro a zero. Esse comando é aplicado ao 
sistema, o sistema reage, o erro é medido de novo, e o ciclo se repete. Por 
isso se chama "malha fechada": a saída do sistema realimenta o próprio 
controlador.

```
┌───────────┐   erro   ┌─────────┐  ação de controle   ┌─────────┐
│ referência│ ───(+)──►│   PID   │ ───────────────────►│ Sistema │
└───────────┘     ▲    └─────────┘                     └────┬────┘
                  │                                         │
                  └───────────── valor medido ──────────────┘
```

### 1.2 Os três termos

O nome "PID" vem da soma de três termos, cada um reagindo ao erro de uma 
forma diferente:

| Termo | O que observa | O que faz |
|---|---|---|
| **P** — Proporcional | O erro **atual** | Aplica uma correção proporcional ao tamanho do erro: quanto maior o erro, mais forte a correção. Reage rápido, mas sozinho deixa um erro residual — nunca "encosta" exatamente no valor desejado. |
| **I** — Integral | O **acúmulo** do erro ao longo do tempo | Vai somando o erro a cada instante. Se existir um erro pequeno e constante que o termo P sozinho não corrige, o termo I vai "empurrando" a correção até esse erro zerar. |
| **D** — Derivativo | A **taxa de variação** do erro | Antecipa para onde o erro está indo. Ajuda a suavizar a resposta e evitar oscilações (o sistema "passar do ponto" e ficar oscilando em torno do valor desejado). |

A saída do controlador é, de forma simplificada, a soma ponderada desses três 
termos:

```
saída = Kp·erro + Ki·∫erro dt + Kd·(d(erro)/dt)
```

Onde `Kp`, `Ki` e `Kd` são os **ganhos** — números que definem o quanto cada 
termo pesa na decisão final. Ajustar esses ganhos (processo chamado de 
*tuning* ou sintonia) é o que define se o sistema responde rápido demais 
(podendo oscilar) ou devagar demais (podendo demorar a atingir o alvo).

### 1.3 PID contínuo vs. discreto

Na teoria de controle "pura", o PID é descrito por uma equação em tempo 
contínuo (com uma integral e uma derivada de verdade). Mas sistemas 
embarcados — como o computador que roda dentro do carro — não processam sinal 
contínuo: eles executam em **passos de tempo fixos** (ex: a cada alguns 
milissegundos). Por isso, na prática, o PID é implementado na forma 
**discreta**: a integral vira uma soma acumulada a cada ciclo, e a derivada 
vira uma diferença entre a amostra atual e a anterior.

### 1.4 Por que filtrar o termo derivativo

O termo derivativo, na prática, é sensível a ruído. Se o sensor que mede a 
grandeza controlada tiver qualquer variação pequena e aleatória (ruído 
elétrico, imprecisão de medição), a derivada desse ruído pode ser grande — 
fazendo o controlador reagir a uma variação que não é real. Por isso, é comum 
adicionar um **filtro** ao termo derivativo, suavizando o sinal antes de 
calcular sua variação. Esse filtro costuma ser representado por um parâmetro 
adicional, geralmente chamado de `N`.

---

## 2. O PID no Sistema do Carro

### 2.1 Onde ele está e o que ele controla

No sistema de controle do veículo, o PID aparece no **subsistema de Controle 
Longitudinal** (dentro do controlador principal do `implementation_deploy.slx`, 
seção 7 do documento de arquitetura). Ele é responsável por controlar a 
**velocidade do carro**.

Diferente de um PID de temperatura ou altitude, aqui a referência (o 
*setpoint*) não é um valor fixo escolhido pelo operador — ela é **calculada 
dinamicamente**, a cada instante, em função da curvatura da trajetória que o 
carro está prestes a percorrer:

```
v_max_curva = √(raio_da_curva × aceleração_lateral_máxima)
```

Ou seja: quanto mais fechada a curva à frente (menor o raio), menor a 
velocidade de referência calculada — e é essa velocidade de referência que o 
PID persegue.

### 2.2 O laço de controle, na prática

```
┌──────────────────--──┐
│  Raio de curvatura   │
│  (calculado à parte) │
└──────────┬───────────┘
           ▼
┌───────────────────==─────┐
│ v_max_curva =            │
│ √(raio × acel_lateral)   │
└──────────┬───────────────┘
           ▼
┌────────────────────==────┐       ┌────-──────────┐
│ Saturação                │       │ Velocidade    │
│ (0 a velocidade máxima   │       │ atual do carro│
│  do carro)               │       │ (vel_x)       │
└──────────┬───────────────┘       └──────┬────────┘
           │                              │
           └──────────────► erro ◄────────┘
                              │
                              ▼
                    ┌────────────────==───┐
                    │   PID Controller    │
                    │ (P, I, D, filtro N) │
                    └──────────┬──────────┘
                               ▼
                    velocidade_desejada
                    (saída para o pedal)
```

### 2.3 Parâmetros configurados no bloco (`implementation_deploy.slx`)

| Parâmetro | Valor no modelo | Papel |
|---|---|---|
| Ganho Proporcional (`Kp`) | `constans_PID(1)` | Reage ao erro de velocidade atual |
| Ganho Integral (`Ki`) | `constans_PID(2)` | Elimina erro residual de velocidade ao longo do tempo |
| Ganho Derivativo (`Kd`) | `constans_PID(3)` | Suaviza a resposta, evita oscilação de velocidade |
| Filtro do derivativo (`N`) | `constans_PID(4)` | Filtra ruído do sensor de velocidade antes de derivar |
| Forma do controlador | Paralela | `Kp`, `Ki`, `Kd` somados de forma independente |
| Domínio | Discreto | Compatível com execução em passos de tempo fixos no processador embarcado |
| Método de integração | Forward Euler | Método numérico usado para aproximar a integral discretamente |
| Aceleração lateral máxima | `aceleracao_lateral_max` (variável) | Define o quão "agressiva" a curva pode ser antes de reduzir a velocidade |
| Velocidade máxima do carro | `max_velocity_longitudinal` (variável) | Limite superior de saturação da referência |

> Os valores numéricos de `constans_PID`, `aceleracao_lateral_max` e 
> `max_velocity_longitudinal` são definidos fora do diagrama (workspace ou 
> script de inicialização do modelo) — os valores atuais devem ser conferidos 
> separadamente.

### 2.4 Por que a saturação vem antes do PID, e não depois

Um ponto de projeto que vale destacar: a saturação de velocidade (`0` a 
`max_velocity_longitudinal`) é aplicada **na referência**, antes dela entrar 
no PID — não na saída do controlador. Isso é intencional: evita que o sistema 
tente perseguir uma velocidade fisicamente impossível (por exemplo, se o 
cálculo do raio de curvatura resultasse em um valor muito alto por causa de 
uma divisão quase por zero em um trecho de trajetória reta). Dessa forma, o 
PID sempre recebe um alvo dentro de limites plausíveis para o veículo.
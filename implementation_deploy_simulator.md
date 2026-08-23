# Sistema de Controle — implementation_deploy

## 1. Visão Geral

Este documento descreve o sistema de controle do carro autônomo implementado no 
modelo `implementation_deploy`, em Simulink/Stateflow. Diferente da versão de 
simulação (baseada inteiramente em tópicos ROS 2), este modelo representa a 
**versão de implantação embarcada**, integrando o barramento **CAN** como meio 
de comunicação com os atuadores e sensores do veículo, além do ROS 2 para 
recebimento da rota desejada.

O sistema é responsável por:

- Receber comandos de habilitação/desabilitação e frenagem do veículo via CAN 
  (`on_off`, `break_cmd`);
- Receber a velocidade atual do veículo via ROS 2 (tópico do sensor inercial);
- Receber a rota desejada (waypoints `x`, `y`) via ROS 2;
- Calcular, a partir dessas informações, os comandos de **direção** (ângulo de 
  esterçamento) e **aceleração** (pedal) necessários para seguir a rota;
- Transmitir esses comandos de volta ao veículo via CAN.

Do ponto de vista teórico, o sistema combina três blocos de controle clássicos 
da robótica móvel/veicular:

1. Uma **máquina de estados** que gerencia o modo de operação do sistema 
   (ativo/inativo);
2. Um **controlador lateral** baseado no modelo cinemático de bicicleta 
   (Stanley Controller), responsável pela direção;
3. Um **controlador longitudinal**, que define a velocidade desejada em função 
   da curvatura da trajetória, garantindo que o veículo não tente fazer curvas 
   fechadas em alta velocidade.

## 2. Arquitetura Geral

Em alto nível, o sistema pode ser dividido em quatro camadas funcionais, que 
se comunicam através de barramentos de sinais (buses) e mensagens CAN/ROS 2:

```
┌──────────────────--────────┐
│   Camada de Entrada        │
│                            │
│  - CAN Receive/Unpack      │
│    (on_off, break_cmd)     │
│  - ROS 2 Subscribe         │
│    (velocidade, rota)      │
└─────────────┬──────────────┘
              │
              ▼
┌--──────────────────────────┐
│  Gerenciamento de Estado   │
│                            │
│  - Máquina de Estados      │
│    (Idle / HV)             │
│    habilita/desabilita     │
│    o controlador           │
└─────────────┬──────────────┘
              │ (habilita)
              ▼
┌--──────────────────────────┐
│      Controlador           │
│                            │
│  - Cálculo do raio de      │
│    curvatura da rota       │
│  - Controle Lateral        │
│    (Stanley Controller)    │
│  - Controle Longitudinal   │
│    (velocidade via PID)    │
└─────────────┬──────────────┘
              │
              ▼
┌──────────────────────────┐
│   Camada de Saída        │
│                          │
│  - CAN Pack/Transmit     │
│    (steering_cmd,        │
│     pedal_cmd)           │
└──────────────────────────┘
```

**Fluxo resumido:** os dados de entrada (estado do veículo via CAN, posição/rota 
via ROS 2) alimentam a máquina de estados, que decide se o controlador deve 
estar ativo. Enquanto ativo, o controlador calcula a curvatura da trajetória 
desejada, usa essa curvatura tanto para definir o ângulo de esterçamento 
(controle lateral) quanto a velocidade segura para a curva (controle 
longitudinal). Os dois comandos resultantes são então codificados e 
transmitidos de volta ao veículo via CAN.

## 3. Interfaces de Comunicação (ROS 2 e CAN)

O sistema se comunica com o restante do veículo através de dois protocolos: 
**ROS 2** (para receber a rota desejada e a velocidade medida) e **CAN** (para 
receber comandos de habilitação/frenagem e para transmitir os comandos de 
atuação). O barramento CAN (*Controller Area Network*) é um protocolo de comunicação que estabelece a ponte entre os modelos Simulink
e os componenetes físicos do carro, dentre eles sensores e atuadores. Ou seja, o CAN nos fornece a interface real de hardware.

### 3.1 Entradas via ROS 2 (Subscribers)

| Tópico | Tipo de Mensagem | Dado Extraído | Uso no Sistema |
|---|---|---|---|
| `/sbg_driver/SbgOdoVel` | `geometry_msgs/Pose2D` | Campo `vel` (velocidade) | `vel_x`, a velocidade atual do veículo |
| `/teste/route` | `geometry_msgs/PoseStamped` | Campos `pose.position.x` e `pose.position.y` | `desired_x` e `desired_y`, o waypoint (ponto de destino) desejado |

Ambos os subscribers utilizam o mesmo tempo de amostragem, definido pela 
variável `ts_ros2`.

### 3.2 Entradas via CAN

| Parâmetro | Valor |
|---|---|
| Dispositivo | `can0` |
| Identificador da mensagem | `0x18ff1080` (extended, 29 bits) |
| Tamanho da mensagem | 8 bytes |
| Período de amostragem | 0,005 s |

Essa mensagem é decodificada (CAN Unpack) em dois sinais:

| Sinal | Posição (bit) | Tamanho (bits) | Faixa de valores | Uso no Sistema |
|---|---|---|---|---|
| `hv_onoff` | 0 | 2 | 0 a 3 | `on_off` — habilita/desabilita o sistema de controle |
| `break_cmd` | 3 | 8 | 0 a 255 | `break` — indica acionamento do freio |

### 3.3 Saídas via CAN

O sistema transmite dois comandos de atuação, cada um como uma mensagem CAN 
independente, através do dispositivo `vcan0`:

| Comando | Sinal Codificado | Identificador | Tamanho | Faixa de Valores |
|---|---|---|---|---|
| Direção (esterçamento) | `steering_cmd` | `0x401` (extended, 29 bits) | 4 bytes | -Inf a Inf (assinado) |
| Aceleração (pedal) | `pedal_cmd` | `13` | 1 byte | 0 a 100 (não assinado) |

Antes de ser empacotado, o sinal de direção passa por um processo de 
escalonamento: o ângulo de esterçamento calculado pelo controlador é 
multiplicado por um ganho fixo (`30555`) e convertido para inteiro, de forma a 
representar o valor em uma unidade compatível com o formato esperado pelo 
barramento CAN do veículo.  
  
## 4. Subsistema: Máquina de Estados (Liga/Desliga)

### 4.1 Propósito

Este subsistema é implementado como um **Stateflow Chart** e tem uma única 
responsabilidade: decidir, a cada instante, se o controlador principal 
do veículo (direção + velocidade) deve estar **ativo ou inativo**. Ele funciona 
como uma camada de segurança/gerenciamento de modo, situada entre a aquisição 
de dados e o controlador propriamente dito.

O sinal de saída desse subsistema (`Hv_on`) não é usado como uma entrada comum 
de dados — ele é conectado à **porta de habilitação (*enable*)** do subsistema 
de controle. Ou seja, o controlador inteiro é um **Enabled Subsystem**: ele só executa sua lógica enquanto `Hv_on = 1`. 
Quando desabilitado, o subsistema simplesmente não é executado naquele passo 
de simulação/execução.

### 4.2 Entradas e Saída

| Sinal | Direção | Origem/Destino |
|---|---|---|
| `on_off` | Entrada | Vem do sinal `hv_onoff`, decodificado da mensagem CAN de entrada (seção 3.2) |
| `break_cmd` | Entrada | Vem do sinal `break_cmd`, decodificado da mesma mensagem CAN |
| `Hv_on` | Saída | Habilita (*enable*) o subsistema de controle (Stanley + velocidade) |

### 4.3 Estados e Transições

O chart possui dois estados:

```
                [on_off == true]
     ┌──────┐ ─────────────────────► ┌──────┐
     │ Idle │                        │  HV  │
     │      │ ◄───────────────────── │      │  entry: Hv_on = 1
     └──────┘   [on_off == false]    └──────┘
                        OU
                [break_cmd == true]
```

| Estado | Descrição |
|---|---|
| **Idle** | Estado inicial (padrão) do sistema. O controlador permanece desabilitado. |
| **HV** *("High Voltage"/habilitado)* | Estado ativo. Na entrada deste estado, a variável `Hv_on` é setada para `1`, habilitando o controlador. |

| Transição | Condição | Efeito |
|---|---|---|
| `Idle → HV` | `on_off == true` | Sistema é ligado; `Hv_on` passa a valer `1` |
| `HV → Idle` | `on_off == false` **OU** `break_cmd == true` | Sistema é desligado — por comando explícito de desligamento |

### 4.4 Interpretação teórica

Essa máquina de estados implementa uma lógica de 
**intertravamento (interlock)**: o controlador de direção e velocidade só pode 
atuar sobre o veículo se o operador/sistema supervisório autorizou 
explicitamente (`on_off == true`).  
  
## 5. Subsistema: Controle Lateral (Stanley Controller)

### 5.1 Propósito

Este subsistema é responsável por calcular o **ângulo de esterçamento** 
necessário para que o veículo siga a trajetória desejada. Ele utiliza o 
**Stanley Controller**, um algoritmo clássico de controle lateral para 
veículos autônomos, baseado no modelo cinemático de bicicleta. (Fazer o curso sobre modelo bicicleta do Coursera)

De forma geral, o Stanley Controller ajusta a direção das rodas combinando 
dois erros: o desalinhamento angular entre o veículo e a trajetória, e a distância entre a posição atual do veículo e o caminho 
desejado. Uma explicação mais aprofundada do funcionamento matemático do 
algoritmo está disponível na aula gravada pela equipe. (Assiste lá no Drive, dá uma força pro nosso trabalho)

### 5.2 Entradas e Saída

| Sinal | Descrição |
|---|---|
| `posicao` (`CurrPose`) | Posição atual do veículo |
| `coossrdenadas_referencias` (`RefPose`) | Ponto de referência da trajetória desejada |
| `Velo passada` (`CurrVelocity`) | Velocidade atual do veículo |
| `direcao` (`Direction`) | Sentido de deslocamento do veículo |
| `steering_cmd` | **Saída**: comando de ângulo de esterçamento |

### 5.3 Parâmetros configurados no bloco

O bloco utilizado é o `Lateral Controller Stanley`, da biblioteca de Driving 
Toolbox, configurado com os seguintes parâmetros:

| Parâmetro | Valor no modelo |
|---|---|
| Modelo do veículo | Kinematic Bicycle Model (modelo cinemático de bicicleta) |
| Ganho de posição (dianteiro) | `stanley_kpfw` |
| Ganho de posição (traseiro) | `stanley_kprv` |
| Ganho de taxa de guinada (*yaw rate*) | `1` |
| Ganho de atraso (*delay*) | `0.2` |
| Massa do veículo | `230` kg |
| Distância eixo-CG dianteiro (`Lf`) | `1.4` m |
| Distância eixo-CG traseiro (`Lr`) | `1.6` m |
| Rigidez do pneu (*tire stiffness*) | `12e3` |
| Entre-eixos (*wheelbase*) | `wheelbase` (variável) |
| Ângulo máximo de esterçamento | `max_steering_angle` (variável) |

### 5.4 Saída do subsistema

O `steering_cmd` calculado por este bloco alimenta diretamente a saída 
`angulo_estercamento` do subsistema, que segue para a camada de comunicação 
CAN (seção 3.3), onde é escalonado e transmitido ao veículo.  
  
## 6. Subsistema: Cálculo do Raio de Curvatura

### 6.1 Propósito

Antes de calcular tanto o comando de direção quanto a velocidade segura para 
a trajetória, o sistema precisa saber **o quão "fechada" é a curva** que o 
veículo está prestes a percorrer. Esse subsistema calcula justamente isso: o 
**raio de curvatura instantâneo** da trajetória desejada, a partir da sequência 
de waypoints recebida (`desired_x`, `desired_y`).

Esse valor de raio (chamado de `curvature` internamente) é usado por dois 
subsistemas mais adiante: o controlador longitudinal, para saber qual a 
velocidade máxima segura na curva (seção 7).

### 6.2 Entradas e Saída

| Sinal | Descrição |
|---|---|
| `desired_x` | Coordenada X do waypoint desejado |
| `desired_y` | Coordenada Y do waypoint desejado |
| `curvature` (`raio`) | **Saída**: raio de curvatura calculado |

### 6.3 Fundamento teórico

O cálculo é baseado na fórmula clássica de **raio de curvatura** para uma 
curva paramétrica discreta:

```
           (dX² + dY²)^(3/2)
   R  =  ─────────────────────
           | dX·d2Y − dY·d2x |
```

Onde:
- `dX`, `dY` são as **derivadas de primeira ordem** (velocidade) da posição 
  desejada, obtidas pela diferença entre a amostra atual e a anterior 
  (`z⁻¹`);
- `d2x`, `d2Y` são as **derivadas de segunda ordem** (aceleração), obtidas da 
  mesma forma a partir de `dX`/`dY`.

Ou seja, o subsistema deriva a trajetória desejada duas vezes (usando atrasos 
unitários — *Unit Delay* — para aproximar as derivadas discretamente) para 
estimar a curvatura local da rota, sem precisar de nenhuma informação de mapa 
ou geometria pré-definida.  
  
## 7. Subsistema: Controle Longitudinal (Velocidade Desejada)

### 7.1 Propósito
A ideia teórica é simples: quanto mais fechada a curva (menor o raio), menor 
deve ser a velocidade máxima segura, para manter a aceleração lateral do 
veículo dentro de um limite aceitável.

### 7.2 Entradas e Saída

| Sinal | Descrição |
|---|---|
| `Raio` | Raio de curvatura da trajetória, calculado no subsistema da seção 6 |
| `Velo passada` (`vel_x`) | Velocidade atual do veículo |
| `velocidade_desejada` | **Saída**: velocidade de referência para o controlador |

### 7.3 Fundamento teórico

O cálculo é feito em duas etapas:

**Etapa 1 — Velocidade máxima segura para a curva.** Usa-se a relação física 
entre velocidade, raio de curva e aceleração lateral máxima permitida:

```
   v_max_curva = √(raio × aceleração_lateral_máxima)
```

Quanto maior o raio (curva mais suave), maior 
a velocidade permitida; quanto menor o raio (curva mais fechada), menor a 
velocidade permitida — sempre respeitando um limite de aceleração lateral 
(`aceleracao_lateral_max`) que o veículo pode suportar sem derrapar/perder 
aderência.

O resultado é então **saturado** entre `0` e a velocidade máxima absoluta do 
veículo (`max_velocity_longitudinal`), garantindo que o valor nunca ultrapasse 
os limites físicos/operacionais do carro, independentemente do resultado da 
fórmula.

**Etapa 2 — Ajuste via controlador PID.** A diferença entre essa velocidade 
máxima segura e a velocidade atual do veículo (`Velo passada`) é calculada e 
alimenta um **Controlador PID discreto**, que gera a velocidade desejada final 
a ser enviada ao controlador de aceleração/pedal.

### 7.4 Parâmetros configurados no bloco

| Parâmetro | Valor no modelo |
|---|---|
| Aceleração lateral máxima | `aceleracao_lateral_max` (variável) |
| Velocidade máxima longitudinal | `max_velocity_longitudinal` (variável) |
| Tipo de controlador | PID, forma Paralela, tempo discreto |
| Ganho Proporcional (P) | `constans_PID(1)` |
| Ganho Integral (I) | `constans_PID(2)` |
| Ganho Derivativo (D) | `constans_PID(3)` |
| Coeficiente do filtro derivativo (N) | `constans_PID(4)` |
| Método de integração | Forward Euler |

### 7.5 Saída do subsistema

O valor de `velocidade_desejada` gerado aqui alimenta a saída 
`velocidade_Desejada` do subsistema de controle, que segue para a camada de 
comunicação CAN (seção 3.3), onde é convertido e transmitido como `pedal_cmd`.  
  
## 8. Parâmetros do Sistema

Esta seção reúne, em um único lugar, todos os parâmetros e variáveis 
identificados ao longo dos subsistemas descritos neste documento. Isso 
facilita tanto a leitura de referência rápida quanto a localização de onde 
cada valor deveria ser ajustado, caso o comportamento do veículo precise ser 
recalibrado.

### 8.1 Variáveis externas (definidas fora do diagrama)

Estes parâmetros **não** têm valor numérico fixo dentro do modelo — eles são 
referenciados por nome e precisam ser definidos externamente (workspace do 
MATLAB, script de inicialização, ou arquivo de dados do modelo). Os valores 
numéricos atuais de cada um não fazem parte deste levantamento e devem ser 
conferidos separadamente com quem mantém esse script.

| Variável | Usada em | Significado |
|---|---|---|
| `ts_ros2` | Seção 3.1 | Tempo de amostragem dos subscribers ROS 2 |
| `stanley_kpfw` | Seção 5.3 | Ganho de posição dianteiro do Stanley Controller |
| `stanley_kprv` | Seção 5.3 | Ganho de posição traseiro do Stanley Controller |
| `wheelbase` | Seção 5.3 | Entre-eixos do veículo |
| `max_steering_angle` | Seção 5.3 | Ângulo máximo de esterçamento permitido |
| `aceleracao_lateral_max` | Seção 7.3/7.4 | Aceleração lateral máxima tolerada em curva |
| `max_velocity_longitudinal` | Seção 7.3/7.4 | Velocidade máxima absoluta do veículo |
| `constans_PID(1..4)` | Seção 7.4 | Vetor com os ganhos `P`, `I`, `D` e `N` (filtro derivativo) do controlador de velocidade |

### 8.2 Parâmetros fixos (configurados diretamente nos blocos)

Estes já vêm com valor numérico definido dentro do próprio modelo:

| Parâmetro | Valor | Usado em |
|---|---|---|
| Massa do veículo | `230` kg | Stanley Controller (seção 5.3) |
| Distância eixo-CG dianteiro (`Lf`) | `1.4` m | Stanley Controller (seção 5.3) |
| Distância eixo-CG traseiro (`Lr`) | `1.6` m | Stanley Controller (seção 5.3) |
| Rigidez do pneu (*tire stiffness*) | `12e3` | Stanley Controller (seção 5.3) |
| Ganho de taxa de guinada (*yaw rate*) | `1` | Stanley Controller (seção 5.3) |
| Ganho de atraso (*delay*) | `0.2` | Stanley Controller (seção 5.3) |
| Ganho de escalonamento do comando de direção | `30555` | Saída CAN — `steering_cmd` (seção 3.3) |
| Limite inferior de saturação (denominador da curvatura) | `0.00001` | Cálculo do raio de curvatura (seção 6.4) |
| Limite superior de saturação (denominador da curvatura) | `10000` | Cálculo do raio de curvatura (seção 6.4) |

### 8.3 Parâmetros de comunicação

Consolidação dos identificadores usados na interface CAN, já detalhados na 
seção 3:

| Mensagem | Direção | Identificador | Dispositivo |
|---|---|---|---|
| Habilitação/Freio (`hv_onoff`, `break_cmd`) | Entrada | `0x18ff1080` | `can0` |
| Comando de direção (`steering_cmd`) | Saída | `0x401` | `vcan0` |
| Comando de pedal (`pedal_cmd`) | Saída | `13` | `vcan0` |  
  
"GG, acabou a documentação do modelo principal do carro"  
By Rafael Coutinho & Lucas Ethics

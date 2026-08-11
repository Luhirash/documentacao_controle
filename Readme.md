---
title: "Controle — Subdivisão de Driverless"

team: "E-Racing Unicamp"

season: "2026"

authors: ["Lucas Hirashima e Rafael Neves"]

version: "2.1"

status: "draft"

created: "2026-07-24"

tags: ["driverless", "eracing", "unicamp", "semana_documentacao"]

---

# Controle & Estado atual — Subdivisão de Driverless
## E-Racing Unicamp · Ciclo 2026

> Documentação referente ao período pós-onboarding redigido por novos membros para fins didáticos. Contém vídeo-aulas e conteúdos dinâmicos para aprendizado.

## Sumário

- [1. Primeiros Passos](#1-primeiros-passos)
- [2. Introdução ao Controle](#2-introdução-ao-mapeamento)
- [3. Conceitos Base](#3-conceitos-base)
- [4. Objetivos e Cenário Atual](#4-objetivos-e-cenário-atual)
- [5. Localização e Mapa Global](#5-localização-e-mapa-global)
- [6. Encerramento](#6-encerramento)
- [7. Referências](#7-referências)

## 1. Primeiros Passos

### 1.1 Abertura
Olá! Seja bem vindo a melhor subdivisão de Driverless! O controle!
Primeiramente, parabéns por ter conquistado seu lugar nessa equipe tão linda, e agora já vamos falar do que você precisa saber para trabalhar com a gente!
Antes de começarmos com matérias de fato(física, dinâmica veicular e computação), vamos preparar todo seu ambiente(enviroment).

### 1.2 Dependências técnicas
Enquanto membro de Controle, faremos uso de algumas tecnologias típicas e imprescindíveis para o desenvolvimento de nossos projetos. Entre elas, algumas precisam de instalação separada ou cuidados especiais. 

**Tecnologias típicas:** Python/C/C++, ROS2, MATLAB(23b em diante): Simulink, Stateflow e System composer.

Caso já tenha essas dependências instaladas, prossiga para o tópico 2.

### 1.3 Sistema Operacional
Para desenvolver seus projetos dentro desta subdivisão e como um membro de Driverless em geral, por favor, utilize Linux. 

Preferencialmente, caso não o possua em sua máquina, opte pela versão 22.04, que casa perfeitamente com a distribuição de ROS2 utilizada por nós no computador de bordo(linda jetson agx xavier). 

Em último caso, se já tiver uma versão Linux em sua máquina e ela esteja em outra versão, trate este problema com a utilização de contêineres através do Docker, uma plataforma que permite executar aplicações que possuem diferentes pré-requisitos (como versão de sistema operacional ou linguagem de programação) sem conflitos.

Recomendamos que, para manter seu sistema operacional de preferência (como o Windows), realize um dual boot em sua máquina. Abaixo, deixamos tutoriais para a instalação do Linux e a utilização do Docker para rodar o ROS2, que explicaremos neste documento.

**OBS**: No tutorial de Linux, certifique-se de baixar a versão 22.04 ao adentrar no site oficial do Ubuntu. Caso tenha problemas para realizar a partição, recomendamos a ferramenta MiniTool Partition Wizard, que resolve problemas de arquivos invisíveis ao particionar o disco.
> Tutorial Linux: https://www.youtube.com/watch?v=bzDDiH2Apac


> Tutorial Docker para ROS2: https://www.youtube.com/watch?v=oix-Qs75O08

### 1.4 Python/C/C++ ----falta falar de c
Grande notícia! Na distribuição Ubuntu, Python já vem instalado nativamente. Inclusive, no Linux, muitas coisas são extremamente facilitadas quando o assunto é instalação de programas ou dependências. Para verificar a versão instalada, rode no terminal:

    python3 --version

Caso não tenha nenhuma base, aqui vai uma playlist introdutória com base de lógica de programação, orientação a objetos (muito importante para lidar com ROS2) e, no primeiro episódio, verificações iniciais do sistema em Linux.

> Playlist referenciada: https://www.youtube.com/watch?v=dzXGwmjpKPk&list=PLiLrXujC4CW3AFaMJyhbObGGlehxNC6BF&index=17 

Para C++, o assunto muda um pouco. Python é uma linguagem interpretada, o que significa que o código-fonte é transformado em algo executável e lido em tempo real. C++, por outro lado, é linguagem compilada, o que exige primeiramente que o código-fonte seja traduzido para linguagem de máquina para que depois torne-se executável. Para termos esse tradutor em nossa máquina, executemos os comandos a seguir:

    sudo apt update

    sudo apt install g++

O primeiro comando atualiza lista de programas em nossa máquina. O segundo instala de fato o pacote principal do C++, incluindo o compilador. Pronto! Temos o básico para conseguir programar nossos sistemas.

### 1.5 ROS2
Agora, cuidaremos da instalação do ROS2 em nossa máquina Linux na versão 22.04 da distribuição Ubuntu. Além disso, explicamos como funciona o ROS2 no contexto da nossa divisão de Driverless, especialmente em Mapeamento. Para conferir com mais clareza cada etapa, assista ao conteúdo abaixo:

> Parte 01 - Instalação do ROS2: https://drive.google.com/drive/folders/17aS0WbSZafMps24pM8jX_G5lp2kk-O29?hl=pt-br

**OBS**: Recomendamos que o vídeo abaixo seja visto após ler o resto do documento, para ter uma clareza do porque é importante o ROS2 aqui.
> Parte 02 - Funcionamento do ROS2: https://drive.google.com/drive/folders/17aS0WbSZafMps24pM8jX_G5lp2kk-O29?hl=pt-br

## 2. Introdução ao Controle
Agora, já falando da nossa subdivisão de controle de fato. Antes de seguir para nossa fundamentação teórica, vamos entender de forma rápida como funciona a dinâmica de todas as subdivisões e a nossa em driverless.

### 2.1 Visão Geral
Em Controle, recebemos as coordenadas já processadas por mapeamento, e de fato, fazemos o carro se locomover até essas coordenadas. Tecnicamente, ajustamos os motores(de tração e steering) para que o carro consiga percorrer a rota traçada por mapping, e para fazer isso, utilizamos uma série de modelos físicos virtuais que simulam o comportamento real do carro.

### 2.2 Arquitetura Atual
Para a comunicação entre os subsistemas de cada subdivisão a fim de construir o sistema como um todo, precisamos ter uma maneira de se comunicar para enviar: 

1. As coordenadas de Percepção para Mapeamento

2. A trajetória de Mapeamento para Controle

3. Os comandos de esterçamento e torque para o carro

4. As informações do carro para a Telemetria

Diante disso, utilizamos o middleware ROS2, que apesar do nome ser Robot Operating System, é um conjunto de ferramentas e bibliotecas que facilita o desenvolvimento de robôs e veículos autônomos através do protocolo de comunicação que ele possibilita. Nele, temos diferentes nós que publicam informações em tópicos e, a partir disso, realizam seus comandos e podem repassar informações. Agora provavelmente seja o momento ideal de você conferir aquele vídeo sobre ROS2 que passamos acima.

Como exemplo de funcionamento, podemos ter a seguinte estrutura:

    Nós:
    - perception_node
    - mapping_node
    - control_node

    Tópicos:
    - /coordinates
    - /waypoint

    Mensagens:
    - std_msgs/msg/Float32MultiArray

Exemplo:

    /coordinates:
    [x1, y1, x2, y2]

    /waypoint (ponto médio entre cones):
    [x_wp, y_wp]

O porquê deste waypoint ficará mais claro na seção de conceitos base, mas tenha em mente que é extremamente importante para a construção da trajetória. Normalmente, temos o eixo x como o que está na frente do carro e y como a lateral. Também é comum ter z como a distância à frente do veículo e x como a posição no eixo lateral.

Pipeline:

    Percepção -> Mapeamento -> Controle

    Telemetria (em Paralelo)
# Tutorial de MATLAB

## Motivação
Este tutorial tem como objetivo explicar a sintaxe básica do MATLAB para futuros bixos da Unicamp E-Racing.

## O que é MATLAB e para que serve?
**MATLAB** (Matrix Laboratory) é uma plataforma de programação e computação numérica para aplicações de engenharia e científicas, tais como Aplicações de Álgebra Linear, Análise de Dados, Processamento de Sinais e Imagens e Sistemas Embarcados.
O MATLAB possui linguagem de programação própria de alto nível conhecida também como MATLAB. A linguagem é construída principalmente em cima de C, C++ e Java. Além disso, o MATLAB é a base do **Simulink**, um ambiente de diagramas de blocos para simulação de sistemas complexos de múltiplos domínios.

## Por que utilizar MATLAB?
O MATLAB é muito utilizado em muitas áreas da Engenharia pela sua alta versatilidade e eficiência em aplicações. Dentro do contexto de um **Fómula SAE**, o MATLAB pode ser muito útil para Modelagem e Simulação de Dinâmica Veicular, Sistemas de Controle e para **Validação de Veículo Autônomo**.

## Sumário

- [Comandos Básicos](#comandos-básicos)
- [Operações Básicas](#operações-básicas)
- [Salvando e Carregando Variáveis](#salvando-e-carregando-variáveis)
- [Algumas Funções Matemáticas e Constantes](#algumas-funções-matemáticas-e-constantes)
- [Um Pouco Sobre o Editor MATLAB](#um-pouco-sobre-o-editor-matlab)
- [Vetores e Matrizes](#vetores-e-matrizes)
- [Plotando Gráficos](#plotando-gráficos)
- [Extraindo Dados de Tabelas](#extraindo-dados-de-tabelas)
- [Arrays Lógicos](#arrays-lógicos)
- [Instruções Condicionais](#instruções-condicionais)
- [Laços de Repetição](#laços-de-repetição)
- [Materiais de Apoio e Aprofundamento](#materiais-de-apoio-e-aprofundamento)

## Comandos Básicos
Para executar comandos básicos basta digitá-los na janela de comando e apertar **Enter**. Os prompts serão indicados por **>>**
## Operações Básicas
### Adição
Ao digitar 7 + 3 por exemplo, o programa retorna ans (answer) = 10

    >> 7 + 3

    ans =

        10
### Subtração
Para subtração vale a mesma lógica

    >> 7 - 3

    ans =

        4
### Multiplicação

    >> 3 * 5

    ans =

        15
### Divisão

    >> 8 / 4

    ans =

        2
### Atribuição
O operador de atribuição é o **=** e ele guarda o valor da expressão na direita na variável do lado esquerdo

    >> x = 9 - 4

    x = 

        5
É possível incrementar uma variável com ela própria ou até realizar a atribuição e omitir seu valor no final
    
    >> x = x + 5;                      % Forma Clássica
    >> x += 2;                         % Operador Composto, equivalente a x = x + 2
    >> x++;                            % Operador de incremento único, equivalente a x = x + 1
A notação ponto-e-vírgula é a responsável por omitir o valor da variável a cada operação enquanto o caractere **%** permite realizar comentários como o **#** em Python ou **//** em C++. Além disso é possível buscar comandos recentes apertando a tecla de **seta para cima**, experimente!

**Atençao!** Em casos que uma atribuição de uma variável depende de outra, ao alterar a variável independente, será neecessário recalcular a dependente de novo.

    >> x = 4;
    >> y = x / 2;                                 % y = 2
    >> x = 6;                                   % modifica-se a variável independente
    >> y

    y = 
                                                % y = 2 se mantém mesmo após x = 6;
        2

    >> y = x / 2                                % atualização da variável dependente

    y =

        3

## Salvando e Carregando Variáveis
No MATLAB, é possível salvar variáveis no Workspace digitando **save** seguido do nome do arquivo

    >> save nome_arquivo.mat

O comando **clear** remove todas as variáveis salvas no workspace

    >> clear

Para carregar as variáveis de novo no Workspace basta digitar **load** seguido do nome do arquivo

    >> load meu_arquivo.mat
Relembrando que para verificar o valor de uma variável basta digitar seu nome na janela de comando!

Com o comando **clc** é possível limpar a janela de comando

    >> clc
Para carregar uma variável específica basta digitar **load** seguido do nome do arquivo com o nome da variável acompanhando. Já para salvar estas variáveis específicas em um novo arquivo .mat basta digitar **save** seguido do nome do arquivo com o nome da variável acompanhando.

    >> load meu_arquivo1 x                % a extensão .mat não acompanha os comandos desta vez!
    >> save meu_arquivo2 x

## Algumas Funções Matemáticas e Constantes
O MATLAB apresenta suporte para algumas funções e constantes matemáticas como:

    >> sqrt(2);                                   % Raíz Quadrada
    >> abs(-9);                                   % Valor Absoluto
    >> exp(x);                                    % Função Exponencial Natural
    >> sin(-1);                                   % Função Seno
    >> cos(5);                                    % Função Cosseno
    >> tan(3/2);                                  % Função Tangente
    >> log(y);                                    % Função Logaritmo Natural
    >> logb(a);                                   % Função Log de a na base b
    >> round(p);                                  % Arredonda para o inteiro mais próximo
    >> floor(p);                                  % Função Piso
    >> ceil(t);                                   % Função Teto
    >> i;                                         % Unidade Básica Imaginária
    >> pi;                                        % Constante Pi (3,1415...)
    >> Inf;                                       % Infinito

## Um Pouco Sobre o Editor MATLAB

### Executando o programa
Na janela de comando do MATLAB é possível separar scripts nas caixas/blocos de comando cinzas. No botão **RUN**, é possível executar todos os scripts de forma coletiva na ordem em que eles estão no programa. Porém é possível rodar cada bloco de código de forma independente selecionando-os e depois exeutando **RUN SECTION**. É importante ressaltar que o bloco de comando só atualiza o valor de uma variável declarada acima dele se o bloco de comando anterior que contém essa variável for executado antes.

### Manipulando Seções
Para criar uma seção no **Live Editor** utiliza-se o botão **Section Break**. Após a criação da seção é possível escrever texto e blocos de código. Para escrever textos basta clicar no botão **Text** para ativar o modo de edição de texto, já para criar um bloco de código deve-se clicar no botão **Code**.

### Debuggando o Código
O Debbuger do MATLAB avisa com um tracejado em vermelho além do caractere "!" (também em vermelho) onde está o erro de sintaxe do seu código. Já os aviso em amarelo são mensagens de cuidado, não necessariamente essa mensagem indica que o código está completamente errado, mas definitivamente não é um aviso para ser ignorado.

## Vetores e Matrizes
Um único número em MATLAB é um escalar, ele representa um array de dimensão 1x1
### Arrays
Arrays podem ser declarados da seguinte forma

    >> y = [7 9]

    y =

        7     9
Ao separar os números com espaço o MATLAB cria um vetor em forma de matriz linha. Para criar um vetor em forma de matriz coluna é necessário adicionar ponto-e-vírgula entre os números.

    >> v = [5; 3]

    v =


        5
        3
Criar uma matriz exige combinar os dois tipos de vetores. Para criar uma matriz de N linhas é necessário seguir este modelo

    >> M = [linha1; linha2; ...; linhaN]

    M =


        linha1
        linha2
        ...
        linhaN
É possível realizar operações nos elementos da matriz na hora de declará-la. Além disso, pode-se utilizar vírgula para separar os elementos de um linha ou até espaçamento com vírgula. Fica a cargo do leitor a melhor maneira de se declarar uma matriz.

    >> A = [3^2, 5/2, 8];
    >> B=[6,11,4.5,1];

### Index em Arrays
Em MATLAB, diferente de outras liguagens de programação tradicionais como Python e C, os index de um vetor começam a partir do número 1 e não do 0. Dado um vetor v para extrair o n-ésimo elemento,

    >> elemento_n = v(n);
    >> ultimo_elemento = v(end);                  % passar end como index acessa o último elemento da lista
    >> elemento_i_j = matriz(i, j);               % Extrai o elemento i,j de uma matriz
Caso apenas uma variável n seja passada como parâmetro para uma matriz, MATLAB retorná o n-ésimo elemento de começando a contar de cima para baixo da primeira coluna. \
\
Para obter uma coluna/lihna inteira emprega-se ":" como um dos parâmetros sobre uma matriz

    >> linha = matriz(i, :);                     % vetor que contém todos os elementos da linha i
    >> coluna = matriz(:, j);                    % vetor que contém todos os elementos da coluna j
    >> linha = matriz(i:k,:);                    % vetor que contém todos os elementos das linha i até a linha k
    >> coluna = matriz(:,j:k);                 % vetor que contém todos os elementos das coluna j até a coluna k
Obter múltiplos elementos bem selecionados de um vetor é recomendado aplicar

    >> [elemento_a, elemento_b] = vetor([a b])   % Sendo a e b os idx dos elementos
Mudar um elemento de um index específico lembra a sintaxe de outras liguagens de alto nível como Python

    >> vetor(n) = 0.5;
    >> matriz(i, j) = 9.6;

### Métodos para Criação de Vetores
Para criar um vetor linha que começa em um número n e vai até p de um em um basta utilizar o operador ":"

    >> V = 1 : 3                                      % Colchetes não são necessários aqui!

    V =

        1     2     3
Também é possível alterar a razão do espaçamento entre os números do vetor, a sintaxe para isso é

    >> U = 20 : 2 :26                                    % primeiro elemento:razão:último elemento

    U =

        20     22     24     26
A função **linspace** (linearly spaced) permite criar um vetor cujo tamanho já é conhecido

    >> w = linspace(0,1,5)                % primeiro elemento,último elemento,quantidade de elementos

    w = 

        0     0.250     0.500     0.750     1.000
A **Transposição de Matrizes** pode ser combinada com os métodos ":" e linspace para a criação de um vetor coluna linearmente espaçado. O operador de transposição é o " ' " (uma aspa simples).

    >> x = 1 : 3

    x =

        1     2     3

    x = x'

    x =

        1
        2
        3
É possível otimizar esse processo realizando

    >> t = (3:2:9)'

    t =

        3
        5
        7
        9
É possível utilizar vetores para criar matrizes, por exemplo

    >> x = [1; 2];
    >> y = [3; 4];
    >> z = [x y]                      % Insere-se vetores coluna 2 x 1 para formar uma matriz 2 x 2

    z =

        1     3
        2     4

    >> a = [1, 2];
    >> b = [3, 4];                   % Insere-se vetores linha 1 x 2 para formar uma matriz 2 x 2
    >> c = [a; b]

    c =

        1     2
        3     4
    
### Criando e Manipulando Arrays com Funções
Para criar uma matriz quadrada de dimensões n x n de números aleatórios utiliza-se o método **rand**

    >> x = rand(2);
O método **rand** também existe para matrizes de dimensões n x m

    >> y = rand(n, m)                                     % rand(linhas, colunas)
Uma matriz nula pode ser criada a partir de uma função parecida com rand, **zeros**. Já uma matriz cujos números são todos 1, pode ser criada com a função **ones**

    >> z = zeros(2, 3);
    >> o = ones(6, 11);
Para obter as dimensões de uma matriz pode ser útil aplicar o método **size**. Levando em conta a matriz z do exemplo anterior,

    >> size(z)

    ans =

          6     11

    >> [n, m] = size(z)                % Atribuindo as dimensões a variáveis

    n =

        6
    m =

        11
A função **max** e **min** no MATLAB serve para encontrar os valores máximo e mínimo em um vetor ou matriz. Além disso, é possível aplicar funções sobre uma matriz executando, assim, esssa função para cada um dos elementos de uma matriz ou vetor e obtendo uma nova matriz com os novos elementos calculados.

    >> xmax = max(V);                                                        % máximo do vetor V
    >> xmin = max(V);
    >> S = sin(M);                       % matriz cujos elementos são os senos dos elementos de M
    
    >> xmax = max(M);                    % Seja M uma matriz, essa função retorna um vetor linha
                                         % contendo os maiores elementos de cada coluna
A função **length** retorna o tamanho/quantidade de elementos de um vetor

    >> t = length(vetor);

### Operações Sobre Vetores
Adicionando um escalar a todos os elementos de um vetor

    >> x = [1, 2, 3];
    >> y = x + 2

    y

        3     4     5
Somar arrays requer que eles sejam de mesma ordem, ou seja, ambos devem ser de dimensões n x m. Para somar arrays então faz-se

    >> v1 = [1, 2, 3, 4, 5];                       % 1 x 5
    >> v2 = [6, 7, 8, 9, 10];                         % 1 x 5
    >> vs = v1 + v2

    vs =

        7     9     11     13     15
Multiplicar ou dividir um vetor por um escalar pedem as seguintes notações

    >> A = [4, 7, 3];
    >> B = 2*A

    B =

        8     14     6
    
    >> C = B/4

    C =

        2     3.50     1.50
O MATLAB suporta dois tipos de multiplicação entre matrizes. O primeiro é a forma tradicional e já formalizada na matemática, em que duas matrizes A e B suportam multiplicação entre si se forem de ordens n x m e m x p respectivamente. Esse processo resulta em uma matriz C de ordem n x p. A outra forma de multiplicar matrizes no MATLAB é multiplicar cada elemento a(ij) da matriz A(n x m) pelo elemento b(ij) da matriz B também n x m. O resultado dessa operação gera uma matriz C de ordem n x m tal que c(ij) = a(ij)*b(ij).

    >> A = [1, 2, 3; 4, 5, 6];                                                       % A(2 x 3)
    >> B = [9, 8, 7; 6, 5, 4; 3, 2, 1];                                                  % B(3 x 3)
    >> C = A * B

    C =                                                                                 % C(2 x 3)

        30     24     18
        84     69     54

    >> D = [6, 5, 4; 3, 2, 1];                                                          % D(2 x 3)
    >> E = A .* D                       % Utiliza-se o operador ".*" para o segundo tipo de multiplicação

    E =

        6    10    12
        12   10    6

## Consultando a Documentação de Funções
O método **doc** abre uma guia que contém a documentação da função seguida do comando doc. Algumas das funções mais utilizadas no MATLAB são: max, min, randi, plot, mean, det e size.

    >> doc plot

## Plotando Gráficos
### Plotando Vetores
Pode-se plotar vetores de mesma quantidade de elementos utilizando a função **plot**. Um dos vetores será responsável por atribuir as coordenadas x dos pontos do gráfico e outro pelas coordenadas y. Por fim, dois vetores se combinarão para formar um gráfico contínuo e, por consequência, talvez aproximados em alguns pontos.

    >> vx = [1, 2];
    >> vy = [2, 4];
    >> plot(vx, vy)                                 % Plotando uma reta de equação y = 2x
### Plotando Pontos
Plotar somente pontos no gráfico requer que os parâmetros sejam escalares na função plot.

    >> plot(vx, vy, "r*")                       % Plota os pontos de vx e vy em forma de um asterisco 
                                                % (*) vermelho (r referente a red)
Caso seja preciso ligar esses pontos de uma forma que não seja por uma linha sólida, será preciso informar o tipo de tracejado

    >> plor(vx, vy, "r--*")                     % "--" informa o tipo de tracejado do gráfico
Para mais formas de estilização consulte [Line Specification](https://www.mathworks.com/help/matlab/ref/plot.html#btzitot_sep_mw_3a76f056-2882-44d7-8e73-c695c0c54ca8). \
\
Comparar gráficos é muitas vezes essencial e, portanto, o MATLAB possui recursos para plotar um gráficos em um mesmo plano com o comando **hold on**. O comando hold on plota todos os gráficos abaixo dele no mesmo plano do gráfico que vem logo antes do hold on. Para desativar essa sobreposição basta digitar **hold off**.

    >> plot(x1,y1)                              % Gráfico 1  
    >> hold on                                  % Plota o gráfico 2 sobre o gráfico 1
    >> plot(x2,y2)                              % Gráfico 2
Plotar apenas um vetor retorna um gráfico que contém cada elemento do vetor no eixo y enquanto o eixo x representa a quantidade de números daquele vetor.

    >> plot(v1)

Para aumentar a espessura do gráfico é necessário acrescentar o parâmetro **LineWidth=x** sendo x a espessura desejada

    >> plot(v1,"y--o",LineWidth=3)               % gráfico de v1 amarelo com linha tracejada e espessura 3
### Fazendo Anotações nos Gráficos
O título de um gráfico tem extrema importância, pois é geralmenete ele que está dizendo o que está sendo analisado. Para escrever o título de um gráfico, é necessário utilizar a função **title** e colocar o título entre aspas. É possível concatenar strings dentro da função título permitindo juntar strings com uma variável.

    >> title("Meu Gráfico")
    >> title("Meu Gráfico nº " + num_experimento)

Para nomear os eixos é preciso utilizar as funções **xlabel** e **ylabel**.

    >> xlabel("Tempo (s)")                               % Eixo x nomeado como Tempo (s)
    >> ylabel("Posição (m)")                             % Eixo y nomeado como Posição (m)

Legendas podem ser feitas com a função **legend**

    >> Legend("Experimento A", "Experimento B")

## Extraindo Dados de Tabelas
MATLAB possui suporte para tabelas e é possível obter os dados registrados nessas tabelas. Para obter um dado e armazená-lo em uma variável faz-se

    >> variavel = nome_tabela.nome_coluna               % Cria um vetor que contém os dados de toda a coluna
### Modificando/Adicionando Dados em Tabelas
Modificar um dado ou adicioná-lo exige o mesmo processo. Se a variável que está recebendo uma atribuição não existe naquela tabela, ela será adicionada, caso exista, será apenas modificada.

    >> nome_tabela.nome_da_coluna = valor
Para extrair uma linha usa-se

    >> variavel = nome_tabela(linha_inicial:linha_final,:)        % Se desejar exibir uma coluna j deve-se
                                                                  % colocar o número j no lugar do último
                                                                  % parâmetro. Os dois pontos extraem todas as colunas

## Arrays Lógicos
Os valores booleanos/lógicos, são valores que resultam de comparações. Assim como na linguagem C, MATLAB não apresenta um tipo booleano, em vez disso adota 0 para **false** e 1 para **true**. Assim qualquer operação lógica como >, <, >=, <=, ==, ~= (equivalente ao != do C) irá retornar 1 ou 0.
### Operações Lógicas Sobre Arrays
É possível realizar operações lógicas entre um array e um escalar, o resultado disso será um array que contém somente 0 ou 1, os quais são resultados de comparações entre cada um dos elementos e o escalar escolhido.

    >> v = [1,2,3];                     % isso vale para todos os operadores lógicos listados anteriormente
    >> y = v > 2

    y =

        0     0     1
### Operadores AND, OR e NOT
Os operadores **AND, OR e NOT** já são bem famosos na computação, mas mesmo já bem consolidados, sempre vale a pena uma revisão. Lembrando que em MATLAB os operadores AND, OR e NOT são escritos como **&, | e ~** respectivamente
### Operador & (AND)
| A | B | A & B |
|------------|------------|-----------|
| 1 | 1 | 1 |
| 1 | 0 | 0 |
| 0 | 1 | 0 |
| 0 | 0 | 0 |

### Operador | (OR)
| A | B | A \| B |
|------------|------------|-----------|
| 1 | 1 | 1 |
| 1 | 0 | 1 |
| 0 | 1 | 1 |
| 0 | 0 | 0 |

### Operador ~ (NOT)
| A | ~A |
|------------|------------|
| 1 | 0 |
| 0 | 1 |

### Index Lógico
Foi visto que é possível criar um vetor lógico a partir de operadores comparativos. Porém não foi visto que vetores lógicos podem ser passados como index para o vetor que criou o vetor lógico. Ao fazer isso, extrai-se os elementos do vetor que indexamente correspondem a 1 (true) para um outro vetor que armazena esses dados.

    >> vetor1 = [3,5,7];
    >> eh_maior = vetor1 > 4                          % Cria-se o vetor lógico

    eh_maior =

                0     1     1

    >> vetor2 = vetor1(eh_maior)                         % Vetor lógico como index

    vetor2 =

                5     7
Esse processo todo, no final, pode ser reduzido para

    >> vetor2 = vetor1(vetor1 > 4);
É possível aplicar para outro vetor 3,

    >> vetor3 = [9, 5, 14];
    >> U = vetor3(vetor1 > 4)

    U =                  % retorna 5 e 14 porque são os números indexamente correspondentes a 1 no vetor lógico

        5     14
Outra aplicação de index lógicos é utilizar para reatribuir valores a vetor original

    >> v1 = [9,1,4];
    >> v1(v1 < 6) = 8                          % Todos os valores de v1 que indexamente corresponderem a 1 no
                                               % vetor lógico, receberão valor 8
    v1 =

         9     8     8                         % Os elementos 1 e 4 retornam 1 (true) para a condição dada
Obs: foram utilizados apenas vetores para demonstrar algumas dessas propriedades, mas vale ressaltar que elas são válidas para qualquer tipo de matriz e aplicação se dá de forma identica.

## Instruções Condicionais
As estruturas condicionais tem grande relevância na computação. Quando um resultado depende de valores de variáveis, então é preciso aplicar algumas estruturas condicionais para moldar nosso código conforme o problema computacional. Os comandos **if/else** são os responsáveis por isso, eles realizam testes lógicos e, se o teste da condição if funciona, retorna 1 e executa o bloco de código da condição if, do contrário, executa o bloco else. A sintaxe para **if/else** em MATLAB é,

    >> if idade >= 18
           disp("Você pode beber e dirigir")
       else
           disp("Você só não pode dirigir")
       end
O comando **end** é obrigatório para encerrar o bloco de decisão. Também, caso haja uma três ou mais cláusulas de decisão, utiliza-se o comando **elseif** para todos os blocos condicionais que não sejam os primeiros. A função **disp** vem de "display" e tem a mesma função que o print do Python, printf do C, cout do C++, etc.

## Laços de Repetição
Quando é preciso executar uma mesma instrução mais uma vez é recomendado utilizar estruturas conhecidas como loops. Cada execução do loop é chamada de iteração e ela pode ser de dois tipos: iteração condicional e iteração limitada. A iteração condicional é realizada com o auxílio do comando **while** que realiza um teste lógico toda vez antes de executar seu bloco de código. Enquanto a condição proposta no bloco while for verdadeira, o loop continua, mas no momento que o teste lógico for falso, o loop se quebra. Já a iteração limitada tem uma quantidade pré-definida de vezes em que ela será executada e ela é acompanhada pelo comando **for** e pode ser muito útil para percorrer elementos de um vetor. A sintaxe para esses dois comandos em MATLAB é,

    >> n = 1;
    >> while n < 5
           disp(n)
           n++;
       end
    
    >> for i 1:5
           disp(i)
       end

## Materiais de Apoio e Aprofundamento
https://matlabacademy.mathworks.com/details/matlab-onramp/gettingstarted \
https://www.ime.unicamp.br/~encpos/VIII_EnCPos/Apostila_Matlab.pdf \
https://www.ic.unicamp.br/~rdahab/cursos/matlab/#material_didatico \
https://mec.ita.br/~adade/Matlab/Web/introduz.htm#Ao%20leitor
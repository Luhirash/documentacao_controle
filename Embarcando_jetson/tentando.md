# Report de Troubleshooting: MATLAB R2023a Add-On Explorer (Exit Code 127)

Olá, Claude! Preciso de ajuda para resolver um problema crítico (Exit Code 127) na instalação de um Add-on no MATLAB R2023a rodando em ambiente Ubuntu Linux (glnxa64). 

Abaixo está todo o contexto do problema e os passos de troubleshooting que já foram realizados. Por favor, analise e sugira o próximo passo.

## 1. Ambiente e Objetivo
* **OS:** Ubuntu Linux (64-bit / `glnxa64`).
* **Software:** MATLAB R2023a (instalado no diretório padrão `/usr/local/MATLAB/R2023a/`).
* **Objetivo:** Instalar o "Simulink Coder Support Package for NVIDIA Jetson" para rodar um modelo `.slx`.

## 2. O Problema
O instalador de Add-ons do MATLAB, que utiliza um navegador embutido baseado em Chromium (CEF), está "crashando" e se recusando a abrir. O problema ocorre tanto ao tentar abrir o "Add-On Explorer" pela interface, quanto ao tentar rodar o pacote offline via linha de comando (`open('~/Downloads/nvidiajetsonandnvidiadrive.mlpkginstall')`).

O erro retornado na *Command Window* é sempre este:

> **Error using matlab.internal.cef.webwindow**
> MATLABWindow application failed to launch. Unable to launch the MATLABWindow application. The exit code was: 127
> 
> Error in matlab.internal.webwindow/createImplementation (line 319)
> `implObj = matlab.internal.cef.webwindow(varargin{:});`
> 
> Error in matlab.internal.addons.AddOnsWindow (line 41)
> `obj.webwindow = matlab.internal.webwindow('about:blank', obj.debugPort);`

## 3. O Que Já Foi Testado (Sem Sucesso)
1. **Instalação de Dependências:** Foram instalados via `apt-get` os pacotes padrão do Chromium (`libnss3`, `libgconf-2-4`, `libxss1`, `libgtk-3-0`, `libgbm1`, etc.).
2. **Aceleração de Hardware:** O MATLAB foi iniciado pelo terminal forçando renderização via software com a flag `/usr/local/MATLAB/R2023a/bin/matlab -softwareopengl`. O erro persistiu.
3. **Checagem de Bibliotecas Dinâmicas (`ldd`):** Executamos o comando `ldd /usr/local/MATLAB/R2023a/bin/glnxa64/MATLABWindow | grep "not found"`. O retorno foi vazio. Não há bibliotecas `.so` ausentes.
4. **Limpeza de Cache:** Excluímos a pasta de cache do Chromium usando `rm -rf ~/.matlab/R2023a/cef_cache`.
5. **Instalação Offline/Manual:** Baixamos o pacote `.mlpkginstall` diretamente do site da MathWorks para evitar a interface de busca. Ao rodar a função `open()` apontando para o arquivo, o sistema invocou o `MATLABWindow` novamente e deu o mesmo Exit Code 127.

## 4. Estado Atual e Suspeita
Como o `ldd` confirmou que não faltam bibliotecas (`.so`), a principal suspeita atual é que o Exit Code 127 está sendo causado por problemas de permissão do Sandbox do Chromium no Ubuntu (relacionado ao `chrome-sandbox` e SUID), ou algum conflito de Wayland/X11. 

A última ação sugerida antes desta mensagem foi tentar executar o `/usr/local/MATLAB/R2023a/bin/glnxa64/MATLABWindow` diretamente no terminal do Ubuntu para forçar a impressão do erro real (stdout/stderr) que o MATLAB está ocultando.

Com base nesse histórico, como devemos proceder para contornar esse Exit Code 127 e conseguir instalar o Support Package da Jetson?
# Troubleshooting: Instalação do Support Package NVIDIA Jetson no MATLAB R2023a

**Projeto:** Fórmula Driverless — nó ROS2 rodando em Jetson AGX Xavier
**Data:** 11-12/09/2026
**Status:** 🟡 Em andamento — volume Docker mal configurado, aguardando correção

---

## 1. Contexto e Objetivo

- **Máquina host:** Ubuntu 24.04 LTS (glnxa64), notebook Lenovo Yoga Slim 7.
- **MATLAB necessário:** R2023a (obrigatório — versão exigida para compatibilidade com o Support Package da Jetson usado no projeto).
- **Objetivo:** Instalar o *"Simulink Coder Support Package for NVIDIA Jetson"* para gerar código a partir de um modelo `.slx` e fazer deploy via SSH em uma **Jetson AGX Xavier**, rodando como nó ROS2 dentro do sistema do carro de Fórmula Driverless.

---

## 2. Problema Original

Ao tentar abrir o Add-On Explorer ou instalar o pacote offline (`open('~/Downloads/nvidiajetsonandnvidiadrive.mlpkginstall')`), o MATLAB retornava:

```
Error using matlab.internal.cef.webwindow
MATLABWindow application failed to launch. Unable to launch the MATLABWindow application.
The exit code was: 127
```

### Causa raiz identificada

**Incompatibilidade de versão do Ubuntu, não sandbox/permissões.**

- O MATLAB R2023a **oficialmente só suporta até Ubuntu 22.04**. Rodando em Ubuntu 24.04, as bibliotecas do sistema (`libstdc++`, `libcairo`, `libfreetype`, `libharfbuzz`) são versões mais novas do que as esperadas pelo componente CEF (Chromium Embedded Framework) usado no `MATLABWindow`.
- O `ldd` não detecta esse problema porque só checa se as `.so` existem, não se os **símbolos** dentro delas são compatíveis. O erro real (mascarado pelo exit code 127) costuma ser um `symbol lookup error` / `undefined symbol`.
- Confirmado oficialmente pela MathWorks: apenas R2024a e R2024b suportam Ubuntu 24.04; para R2023a, a recomendação oficial seria migrar de versão — mas isso não é viável no nosso caso pela dependência do Support Package da Jetson.

### O que já foi testado e **não resolveu** (no host puro, sem container)

| Tentativa | Resultado |
|---|---|
| Instalar dependências padrão do Chromium via `apt-get` (`libnss3`, `libgconf-2-4`, `libxss1`, `libgtk-3-0`, `libgbm1`) | ❌ Erro persistiu |
| Forçar renderização por software (`matlab -softwareopengl`) | ❌ Erro persistiu |
| Checar libs ausentes com `ldd .../MATLABWindow \| grep "not found"` | ❌ Retorno vazio — não detecta incompatibilidade de símbolos |
| Limpar cache do CEF (`rm -rf ~/.matlab/R2023a/cef_cache`) | ❌ Erro persistiu |
| Baixar o `.mlpkginstall` manualmente do site e rodar `open()` direto | ❌ Mesmo erro (exit 127) |
| Tentar `matlab.addons.install(...)` como alternativa via linha de comando sem GUI | ❌ **Não é opção**: esse comando só suporta arquivos `.mltbx`; pacotes de hardware (`.mlpkginstall`) exigem obrigatoriamente o instalador gráfico (CEF) |

---

## 3. Decisão: Isolar o R2023a em um Container Docker

Em vez de perseguir `LD_PRELOAD` símbolo por símbolo no host (solução frágil, e o CEF seria invocado de novo mais adiante, ex: tela de "Hardware Setup" do Simulink Coder), optamos por rodar o MATLAB R2023a dentro de um container com o Ubuntu que ele realmente suporta.

- Usamos a imagem oficial da MathWorks: **`mathworks/matlab:r2023a`** (Docker Hub), que já vem com o Ubuntu/bibliotecas corretas para essa versão.
- Isso **resolveu o exit code 127** — o CEF passou a inicializar corretamente dentro do container.

### `Dockerfile` usado

```dockerfile
# Usa a imagem oficial do R2023a já pré-configurada
FROM mathworks/matlab:r2023a

# Muda temporariamente para root para instalar ferramentas extras úteis
USER root

# Instala pacotes essenciais para deploy na Xavier e compilação
RUN apt-get update && apt-get install -y \
    openssh-client \
    build-essential \
    iputils-ping \
    git \
    net-tools \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Volta para o usuário padrão do container ('matlab')
USER matlab
```

---

## 4. Segundo problema: "o MATLAB abre mas não mostra nada"

A imagem `mathworks/matlab:r2023a` **não abre uma janela local** — ela sobe um servidor gráfico dentro do container, acessível de 3 formas:

1. **Navegador** (recomendado para uso remoto/via SSH) — flag `-browser` + porta mapeada.
2. **noVNC via navegador** — porta 6080.
3. **Cliente VNC** — display `:1` do container, senha padrão `matlab`.

Se o container for iniciado sem nenhuma dessas configurações (sem `-p`, sem `-e DISPLAY`), ele fica "rodando" sem dar nenhum feedback visual — foi exatamente o que aconteceu.

### Comando correto (modo browser)

```bash
docker run -it --rm \
  --name matlab_driverless \
  -p 8888:8888 \
  --shm-size=512M \
  -v /caminho/real/do/projeto:/home/matlab/Documents/MATLAB/projeto_driverless \
  mathworks/matlab:r2023a -browser
```

Acesso: `http://localhost:8888` (ou via túnel SSH `ssh -L 8888:localhost:8888 usuario@host` se a máquina for remota).

✅ **Isso resolveu o problema de "não abre nada".**

---

## 5. Terceiro problema (em aberto): `.slx` não aparece no navegador

Ao clicar em "Open" dentro do MATLAB (rodando no navegador), o arquivo `.slx` da máquina local não aparece — porque o container só enxerga o **próprio filesystem**, isolado do host, a menos que a pasta seja explicitamente montada como *volume* (`-v`).

### Diagnóstico feito

```bash
docker exec -it friendly_wilson ls -la /home/matlab/Documents/MATLAB/projeto_driverless
```
```
total 8
drwxr-xr-x 2 root   root   4096 Sep 12 02:33 .
drwxr-xr-x 1 matlab matlab 4096 Sep 12 02:33 ..
```

**Resultado:** pasta **vazia**, dono `.` = `root:root` (diferente de `..` = `matlab:matlab`). Isso é o padrão de quando o Docker cria automaticamente uma pasta vazia porque **o caminho do host apontado no `-v` não existia de verdade** (provavelmente o placeholder do comando de exemplo não foi substituído pelo caminho real, ou houve erro de digitação no caminho).

### Erro de sintaxe encontrado no caminho

Também identificamos que o comando `docker exec -it <friendly_wilson> ls ...` (copiado com os sinais `< >` de um exemplo/placeholder) falhava com `bash: friendly_wilson: Arquivo ou diretório inexistente`, porque o bash interpretou `<friendly_wilson` como redirecionamento de entrada. **Os `< >` eram só indicação de "substitua aqui" e não deveriam ser digitados.**

---

## 6. ✅ O que funcionou até agora

- [x] Identificar que a causa do exit 127 era incompatibilidade Ubuntu 24.04 × R2023a (não sandbox/permissões).
- [x] Descartar `matlab.addons.install` como bypass (não serve para `.mlpkginstall`).
- [x] Rodar MATLAB R2023a em container Docker com a imagem oficial `mathworks/matlab:r2023a` — **CEF funcionando, sem exit 127**.
- [x] Acessar a interface do MATLAB via navegador (`-browser`, porta 8888).
- [x] Diagnosticar corretamente, via `docker exec ... ls -la`, que o volume estava montado num caminho vazio/errado.

## ❌ O que ainda não funciona

- [ ] Volume do projeto (`-v`) ainda não está apontando para o caminho real onde está o `.slx` no host.
- [ ] Instalação do Support Package da Jetson ainda **não foi concluída** (paramos na etapa de conseguir ver o `.slx` dentro do MATLAB).
- [ ] Deploy via SSH para a Jetson AGX Xavier ainda não foi testado a partir de dentro do container (rede do Docker precisa ser configurada, provavelmente `--network host` ou mapeamento de porta SSH).

---

## 7. Próximos Passos (para retomar)

1. **Achar o caminho real do `.slx`** no host:
   ```bash
   find ~ -iname "*.slx" 2>/dev/null
   ```
2. **Parar o container atual** e subir de novo com o caminho correto (sem placeholders!):
   ```bash
   docker stop matlab_driverless
   docker run -it --rm \
     --name matlab_driverless \
     -p 8888:8888 \
     --shm-size=512M \
     -v /caminho/real/encontrado/pelo/find:/home/matlab/Documents/MATLAB/projeto_driverless \
     mathworks/matlab:r2023a -browser
   ```
3. **Confirmar o volume** antes de abrir o navegador:
   ```bash
   docker exec -it matlab_driverless ls -la /home/matlab/Documents/MATLAB/projeto_driverless
   ```
   Deve listar o `.slx` (e não aparecer mais como pasta vazia `root:root`).
4. Se aparecer erro de permissão ao abrir o arquivo dentro do MATLAB, testar:
   ```bash
   chmod -R a+rw /caminho/real/do/projeto
   ```
5. Dentro do MATLAB no navegador, navegar até `/home/matlab/Documents/MATLAB/projeto_driverless` e abrir o `.slx`.
6. **Instalar o Support Package da Jetson**:
   ```matlab
   open('/home/matlab/Documents/MATLAB/projeto_driverless/nvidiajetsonandnvidiadrive.mlpkginstall')
   ```
   (ajustar caminho conforme onde o `.mlpkginstall` for montado).
7. Após a instalação, testar conectividade SSH com a Jetson **de dentro do container**:
   ```bash
   docker exec -it matlab_driverless ssh usuario@ip_da_jetson
   ```
   Se falhar, provavelmente será necessário rodar o container com `--network host` ou mapear a porta 22 corretamente.
8. Confirmar, dentro do MATLAB, que o alvo de hardware "Jetson AGX Xavier" aparece disponível em **Hardware Setup** do Simulink Coder.
9. Gerar código e fazer deploy do modelo `.slx` como nó ROS2 na Jetson.

---

## 8. Referências e comandos úteis coletados

- Repositório oficial de imagens Docker da MathWorks: [`mathworks-ref-arch/container-images`](https://github.com/mathworks-ref-arch/container-images)
- Imagem usada: [`mathworks/matlab:r2023a`](https://hub.docker.com/r/mathworks/matlab) (Docker Hub)
- Requisitos de sistema Linux do MATLAB: https://www.mathworks.com/support/requirements/matlab-linux.html
- `matlab.addons.install` só aceita `.mltbx` — confirmado em discussões da comunidade MATLAB Central (não serve para `.mlpkginstall` de hardware support packages).

### Comandos de diagnóstico rápido (guardar para próxima sessão)

```bash
# Ver containers rodando e seus nomes
docker ps

# Ver conteúdo de uma pasta dentro do container
docker exec -it <nome_do_container> ls -la <caminho>

# Achar o .slx no host
find ~ -iname "*.slx" 2>/dev/null

# Subir o container com volume + modo browser
docker run -it --rm \
  --name matlab_driverless \
  -p 8888:8888 \
  --shm-size=512M \
  -v /caminho/real/do/projeto:/home/matlab/Documents/MATLAB/projeto_driverless \
  mathworks/matlab:r2023a -browser
```

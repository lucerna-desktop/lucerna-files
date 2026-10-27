# Explorador de arquivos do Lucerna

**Data:** 2026-10-27
**Repositório:** [`lucerna-desktop/lucerna-files`](https://github.com/lucerna-desktop/lucerna-files)
**Situação:** desenho para aprovação.
**Precedentes lidos (só leitura):** `lucerna-sdk/docs/ecossistema.md` e `docs/superpowers/specs/2026-10-19-protocolo-design.md`; `lucernad` (crates, `lucernad-core`, módulos `clipboard`, `power` e `system`, CI); `lucerna-shell` (`core/theme/ThemeManager.qml`, `core/config/Config.qml`, `services/Lucernad.qml`, `services/FileSearch.qml`, `core/panels/Panels.qml`, Pendências); `lucerna-notes` e `lucerna-routine` (só `docs/proposta.md`, ainda sem código); o perfil da org.

Quem decidiu cada coisa está marcado: **(usuário)** ou **(assistente)**. O que não tem marca é consequência direta de uma decisão marcada ou de um precedente citado.

## Por quê

O Lucerna tem shell, daemon e apps em Rust, mas o usuário ainda abre arquivos com um explorador de fora, que ou é leve e limitado (Thunar, PCManFM) ou é completo e puxa o KDE ou o GNOME inteiro (Dolphin, Nautilus). Nenhum deles conversa com o shell: o progresso de uma cópia grande some quando a janela fecha, o launcher busca arquivos por um script (`fd`/`find`), o pendrive não aparece em lugar nenhum e o tema não acompanha o papel de parede.

O `lucerna-files` junta o que cada explorador tem de melhor, sem o peso de nenhum, e é integrado ao shell desde o desenho. É também **o primeiro app do ecossistema com GUI própria**, e por isso define o padrão de GUI que os outros vão seguir.

## Decisões do usuário

- **App do ecossistema Lucerna**, repositório próprio na org `lucerna-desktop`, com as mesmas regras dos outros.
- **Núcleo em Rust compartilhado; TUI em Rust; GUI em Qt.** Extremamente leve e rápido, assíncrono, com muitos recursos integrados e, acima de tudo, integrado ao shell.
- **Inspirações e o que vem de cada uma ("vamos incluir tudo"):**
  - **Thunar/PCManFM:** abre na hora, pouca RAM, sem puxar dependências do KDE nem do GNOME (um limite que nenhum recurso pode furar); a renomeação em massa forte do Thunar (padrões, numeração, prévia antes de aplicar, desfazer).
  - **Nautilus:** simples à primeira vista, com o poder atrás de atalhos e menus; busca rápida, no núcleo, assíncrona.
  - **Dolphin:** abas, painel dividido, terminal embutido, pré-visualização, personalização; nuvem por montagens que o sistema entende (ex.: rclone), sem depender do KDE.
  - **Nemo:** menu de contexto rico, propriedades detalhadas, painel duplo integrado.
  - **Krusader/Double Commander:** painel duplo de verdade, muito teclado; fila de operações grandes com pausa, retomada e conflitos.
  - **Ranger:** atalhos estilo Vim (opcionais), visão em colunas (Miller), pré-visualização de imagens também na TUI.
  - **Evitar:** visual austero, a curva de aprendizado do Krusader, plugins pesados, dependências pesadas.
- **Integração com o shell:** usa o tema e as cores do Lucerna; o progresso das operações longas aparece na central do shell mesmo com o explorador fechado; o launcher busca arquivos pelo mesmo núcleo; pendrives e discos aparecem no shell e abrem no explorador; arquivos copiados entram no histórico da área de transferência.
- **Estrutura:** núcleo (biblioteca Rust assíncrona sobre tokio), TUI (ratatui) e GUI (Qt Quick/QML ligada ao Rust pelo cxx-qt, a mesma linguagem de interface do shell, para reaproveitar o tema).
- **A fila segue o padrão dos outros apps:** o explorador tem **o próprio serviço** (`lucerna-files serve`), e não um módulo no `lucernad`. O serviço é o **dono único** da fila e do diário de operações, sobe sob demanda e sai por ociosidade, como no ecossistema, e atende a TUI, a GUI, a linha de comando e o shell, que mostra o progresso na central pelo mesmo protocolo dos outros apps. **O `lucernad` não depende do `lucerna-files`.**
- **A GUI em Qt é o padrão do ecossistema:** os outros apps (notes, routine, phone) poderão ganhar uma GUI com o mesmo código e a mesma organização. O explorador é o primeiro a seguir esse padrão (seção "Padrão de GUI do ecossistema").
- **Ordem:** núcleo + TUI primeiro, depois a GUI, por último a integração com o shell.
- **Não se limitar à lista:** incluir ideias e recursos a mais que façam sentido (seção "Recursos", marcados por origem).
- **Funciona sozinho** (vale para todo app do ecossistema): instalar e usar o explorador não exige o shell, o `lucernad`, o Hyprland nem o systemd, em qualquer distribuição e ambiente (KDE, GNOME etc.). A integração com o shell é um extra que liga quando ele está presente. O `lucerna-sdk` entra só como biblioteca, na compilação. Ver "Uso sem o Lucerna".

## Decisões do assistente

- **Nome `lucerna-files`** (binários `lucerna-files` e `lucerna-files-gui`, socket `files.sock`, pastas `files`). Confirmado: o repositório já existe, público, espelhando o `lucerna-notes`.
- **Fases:** v1 núcleo + serviço + TUI; v2 GUI (e o padrão de GUI no SDK); v3 shell e portal; v4 os extras, por prioridade.
- **Desfazer sobrescritas:** antes de sobrescrever, o arquivo antigo vai para a lixeira freedesktop, inclusive numa extração por cima. Uma mudança de permissões em massa guarda o modo (e o dono) antigo de cada arquivo no diário.
- **A GUI é um processo único com várias janelas**, por causa do portal FileChooser, do `FileManager1` e do nome único no D-Bus. "Abrir na GUI" com a GUI já aberta abre uma janela nova nesse processo.
- **O shell grava o tema resolvido num JSON público**, com o formato definido num spec pequeno do lado do shell.
- **Onde mora a parte comum da GUI:** no `lucerna-sdk` (seção "Padrão de GUI do ecossistema").
- As demais decisões marcadas *(assistente)* ao longo do texto.

## Precedentes seguidos

| Precedente | Onde | Como entra aqui |
|---|---|---|
| Estrutura núcleo + serviço + TUI + painel; TUI e painel são clientes do serviço; ninguém mexe nos dados por fora | `ecossistema.md` | Igual, com a GUI como mais um cliente. O serviço é o único que escreve na fila, no diário e em `~/.local/share/lucerna/files/`. |
| Protocolo JSON-RPC 2.0 por socket Unix, tópicos com retrato, `shell.sock`, manifesto JSON, ativação por socket do systemd do usuário, saída por ociosidade com trabalho pendente | spec do protocolo, `lucerna-service` | `lucerna-files serve` usa o `lucerna-service` sem mudança no modelo; a fila conta como trabalho pendente. |
| Itens que **pertencem à conexão** no `shell.sock` (barra, não perturbe), reaplicados na reconexão, contados como trabalho pendente | spec do protocolo | O progresso na central é um item novo do mesmo tipo (`progress.set`/`progress.clear`); ver "O que muda no lucerna-sdk". |
| Um binário por app, `lucerna-<app>`, com `tui` e `serve` ocultos, CLI pelo `lucerna-cli`, `--json` e códigos 0/1/2/64 | spec do protocolo, `lucerna-hello` | `lucerna-files` é TUI + CLI + serviço. A GUI é um segundo binário, `lucerna-files-gui` (a TUI não pode ligar à libQt); a regra entra no `ecossistema.md`. |
| Pastas XDG por app (`lucerna-paths`) | `ecossistema.md` | Configuração em `~/.config/lucerna/files.toml`; dados em `~/.local/share/lucerna/files/`; cache em `~/.cache/lucerna/files/`. |
| Workspace Cargo `crates/*`, edição 2024, `rust-toolchain.toml` 1.95, GPL-3.0-only, `unsafe_code = "deny"`, clippy `-D warnings` | `lucerna-sdk`, `lucernad` | Igual. O `unsafe` gerado pelo cxx-qt fica confinado ao crate da GUI, com `allow` só nele. |
| CI: `fmt`, `clippy`, `test` em matriz, ações fixadas por SHA, cron semanal | `lucernad/.github/workflows/ci.yml` | Igual, mais `deps` (dependências proibidas) e `gui` (Qt). |
| Laço supervisionado, publicar só o que mudou, trabalho com prazo numa thread solta, D-Bus por `zbus` com prazo em toda chamada, traits com falsos na feature `testing` | `lucernad-core`, `lucernad-power/bus.rs` | O serviço e as montagens por udisks2 seguem o mesmo molde (copiado, não importado: o `lucerna-files` não depende do `lucernad`). |
| Serviços do shell como adaptadores (`services/*.qml` mantêm as propriedades) | Pendências do shell | `services/FileSearch.qml` deixa o `file-search.sh` e passa a pedir ao `files.sock`; `services/FileOps.qml` novo para a central. |
| Histórico do Wayland com o tipo `files` (`text/uri-list` e `x-special/gnome-copied-files`) | spec da área de transferência do `lucernad` | Copiar no explorador já põe os arquivos no histórico; nada novo no módulo `clipboard`. |
| Discos lidos sem processo externo | spec do `system` do `lucernad` | O aviso de pendrive no shell vem de um módulo do `lucernad` que não depende do explorador (ver "Discos no shell"). |
| Tema como dado, tokens em grupos (cores, fontes, raios, espaçamentos, movimento) | `ThemeManager.qml` | O módulo de tema da GUI tem os mesmos nomes de tokens. |
| Script de medição no repositório | `lucerna-shell/dev/bench-blur.sh` | `dev/bench.sh` mede as metas de desempenho. |
| README com "Situação", `docs/proposta.md`, templates de issue e PR, ruleset da `main` | `lucerna-notes`, `lucerna-routine` | Igual. |

### Onde o desenho sai do precedente

1. **GUI própria com cxx-qt.** Nenhum repositório usa cxx-qt; o shell é Quickshell (C++/QML) e os outros apps só têm painel QML dentro do shell. Por decisão do usuário, a GUI em Qt vira padrão do ecossistema; o que se reaproveita do shell é a linguagem e os **nomes dos tokens**, não os componentes, que importam `Quickshell` e `qs.*` e não carregam fora dele.
2. **Dois binários por app quando há GUI** (`lucerna-<app>` e `lucerna-<app>-gui`).
3. **Um tipo novo de item no `shell.sock`** (o progresso), no mesmo molde da barra.

## Objetivos

- Um explorador que abre na hora, gasta pouca memória e não trava com pasta enorme, disco lento ou montagem morta.
- O mesmo comportamento na TUI e na GUI: as mesmas ações, os mesmos atalhos, a mesma configuração.
- Operações de arquivo seguras: nada se perde numa queda, num conflito ou num cancelamento; tudo se desfaz.
- Integração com o shell: tema, central, launcher, discos, área de transferência, arrastar e soltar, seletor de arquivos do sistema.
- Poder atrás de atalhos e menus, com a primeira tela simples.
- Um padrão de GUI que os outros apps copiem sem decidir de novo.
- pt-BR primeiro, acessível por teclado e por leitor de tela (GUI).

## Não objetivos

- Indexador sempre ligado no estilo Baloo/Tracker. A busca varre sob demanda, com índice opcional e pequeno (v4).
- Plugins com runtime próprio (Python, Lua, JS). A extensão é por ações personalizadas (scripts) e configuração.
- Editor de texto, visualizador de imagem ou player completos: a pré-visualização é para olhar; abrir é com o app padrão.
- Sincronização própria com nuvem: a nuvem chega por montagens (rclone, sshfs).
- MTP por gvfs. Celular, quando houver, pelo `lucerna-phone` (v4).
- Outros sistemas além do Linux; o X11 não é testado.
- Unificar agora os componentes da GUI com os do shell (ver "Padrão de GUI", futuro).

## Anti-objetivos

- **Depender do KDE, do GNOME ou do gvfs** (KIO, KF6, GLib/GIO, GTK, gvfs, tracker). Nenhum recurso justifica furar isso; um job do CI recusa.
- **Exigir qualquer outra peça do ecossistema instalada** (o shell, o `lucernad`) ou o Hyprland e o systemd. Sem elas, o explorador funciona inteiro; a ausência do shell é silêncio, nunca erro nem espera.
- **GUI lenta para abrir** (splash, carregar tudo antes da primeira pintura, QML não compilado).
- **TUI e GUI divergirem** (uma ação existir só numa sem motivo de mídia, atalhos diferentes, configurações separadas).
- **Perder dados numa operação** (sobrescrever sem perguntar nem guardar o antigo, mover entre discos apagando antes de confirmar a cópia, apagar definitivo por engano, deixar arquivo pela metade com o nome final).
- **Dois processos escrevendo na mesma fila ou no mesmo diário.**
- Travar a interface esperando o disco (um `stat` num NFS morto não pode congelar a janela).
- Seguir links em laço ou atravessar sistemas de arquivos sem querer numa busca ou no uso de disco.
- Telemetria de qualquer tipo.
- A TUI ligar à libQt ou a qualquer biblioteca gráfica.
- Lógica de negócio em QML/JS.

## Restrições

### Desempenho (metas mensuráveis; *assistente*)

Medidas por `dev/bench.sh` numa máquina de referência (a do usuário), com o cache de disco quente, salvo quando dito.

| Medida | Meta |
|---|---|
| TUI: do comando até a primeira tela, pasta de 1 mil itens | < 30 ms |
| TUI: memória residente (RSS) parada numa pasta comum | < 15 MB |
| GUI: do comando até a primeira janela pintada | < 250 ms (cache quente), < 500 ms (frio) |
| GUI: nova janela com o processo já aberto | < 80 ms |
| GUI: RSS parada, uma janela, pasta comum | < 80 MB (alvo 60 MB; referência: Thunar) |
| Serviço: da ativação até responder ao primeiro pedido | < 30 ms |
| Serviço: RSS sem operação | < 10 MB |
| Pasta com 100 mil arquivos: primeiros itens na tela | < 50 ms |
| Pasta com 100 mil arquivos: lista completa e ordenada | < 1 s em SSD, sem quadro acima de 16 ms na GUI e sem tecla perdida na TUI |
| Trocar de pasta já vista (cache) | < 16 ms |
| Busca por nome na pasta pessoal (~500 mil arquivos): primeiros resultados | < 100 ms; cancelada a cada tecla |
| Cópia no mesmo disco | ≥ 95% da vazão do `cp`; reflink quando o sistema de arquivos permite |
| Progresso para o shell e os clientes | no máximo 4 atualizações por segundo, só o que mudou |

- Tudo que toca o disco é assíncrono ou vai para threads de bloqueio com prazo; nada na thread da interface.
- Listagem em lotes (streaming), ordenação incremental, ordem natural (`arquivo2` antes de `arquivo10`).
- Miniaturas e pré-visualizações com concorrência limitada, canceladas ao sair da pasta, nunca decodificando imagem grande na interface.
- QML compilado antes (qmlcachegen/AOT) e embutido no binário; componentes carregados sob demanda (`Loader`); nada de módulos Qt pesados.

### Dependências

- **Permitido:** tokio, crates Rust puros sempre que houver; Qt 6 só QtCore, QtGui, QtQml, QtQuick, QtQuick.Controls (estilo Basic, pelos templates) e QtSvg; zbus; ratatui + crossterm; `ratatui-image` (kitty, sixel e blocos como reserva).
- **Proibido:** GLib/GIO/GTK, KF6/KIO, gvfs, tracker, QtWebEngine, QtMultimedia, Python embutido.
- **Ferramentas externas só opcionais e detectadas** (nunca obrigatórias): `rclone`, `sshfs`, `ffmpegthumbnailer`, `pdftoppm`, `git`. Sem elas, o recurso some com uma explicação, sem erro.
- O job `deps` do CI falha se `cargo tree` ou `ldd` dos binários mostrar algo da lista proibida, ou se o binário `lucerna-files` ligar à libQt.

### Testes

- **Não tocam hardware real, D-Bus real nem internet.** udisks2, o portal, o `FileManager1`, o Wayland e o `shell.sock` ficam atrás de traits com falsos na feature `testing` (molde do `lucernad-power`). Montagens FUSE, rclone e sshfs são testados por falsos do executor de comandos.
- Sistema de arquivos real **só em pastas temporárias** (`tempfile`), com `TMPDIR` curto no CI (limite de caminho do socket Unix).
- **Serviço de ponta a ponta** num socket temporário (molde do `lucerna-hello`): dois clientes enfileirando ao mesmo tempo, assinatura da fila, saída por ociosidade só sem trabalho, subida pela linha de comando sem systemd.
- **Segurança de dados:** queda no meio da cópia (matar o serviço e retomar pelo diário), conflito, sobrescrita indo para a lixeira, disco cheio (falso), cancelamento, movimento entre "discos" (falso de `EXDEV`), lixeira em mídia removível (`.Trash-<uid>`), links em laço, desfazer de permissões em massa.
- **Renomeação em massa:** testes de propriedade (nenhuma colisão aplicada; desfazer devolve o estado; troca circular `a→b`, `b→a` funciona).
- **Paridade TUI/GUI:** o catálogo de ações é um só e um teste confere que toda ação tem atalho e entrada na paleta nas duas vistas.
- **GUI:** testes dos ViewModels em Rust (sem janela) e testes QML dos componentes no CI com `QT_QPA_PLATFORM=offscreen`; teste manual no Hyprland aninhado em Docker do shell.
- **Desempenho:** `dev/bench.sh` não roda no CI (máquina variável); roda antes de cada release e quando um PR mexe em listagem, fila ou abertura, com o resultado no PR.

## Arquitetura

### Peças

```
lucerna-files/
├── crates/
│   ├── lucerna-files-core/   Model: biblioteca assíncrona (tokio), sem interface
│   ├── lucerna-files-app/    ViewModel: abas, painéis, seleção, ações, atalhos (sem ratatui nem Qt)
│   ├── lucerna-files/        binário: TUI (ratatui), CLI (lucerna-cli) e `serve` (o serviço)
│   └── lucerna-files-gui/    binário: View Qt (ponte cxx-qt + qml/)
├── data/                     manifesto, .desktop, .portal, serviços D-Bus, unidades systemd
├── dev/bench.sh
└── docs/
```

**`lucerna-files-core`** (feature-first + core + share):

- `core/`: abstração do sistema de arquivos (trait `Fs` com a implementação real e uma falsa), observação de mudanças (inotify, com coalescência), prazos, erros, tipos MIME (shared-mime-info lido direto, sem GLib), caminhos (`lucerna-paths`).
- `share/`: tipos comuns (entrada, URI, tamanho, permissão, eventos) e a serialização para o protocolo.
- Uma pasta por recurso: `listing`, `search`, `ops` (fila, diário, conflitos, desfazer), `trash`, `rename`, `archive`, `thumbnails`, `preview`, `mounts`, `tags`, `bookmarks`, `frecency`, `openwith`, `actions`, `properties`, `checksums`, `diskusage`, `duplicates`, `compare`, `git`, `config`.

**`lucerna-files-app`** é o ViewModel comum: estado de navegação (abas, painéis, pasta, seleção, filtro, ordem, visão), o **catálogo de ações** (id, nome em pt-BR, atalho padrão e Vim, se vale em seleção, se é destrutiva) e o despacho, para o núcleo (leituras) ou para o serviço (mudanças). A TUI e a GUI são vistas finas sobre ele: uma ação nova entra aqui, uma vez, e as duas a ganham.

### O serviço (`lucerna-files serve`) **(usuário)**

- **Dono único** da fila de operações, do diário, da frecência, das conexões de rede montadas pelo explorador e da busca para o launcher. Toda **mudança** em arquivos (copiar, mover, renomear, lixeira, apagar, criar pasta, permissões, compactar, extrair, montar) passa por ele, inclusive as pequenas (renomear um arquivo): é o que dá o desfazer e o diário completos. *(assistente)*
- **Leituras ficam no processo da vista** (listar, pré-visualizar, buscar na pasta aberta): ir e voltar pelo socket a cada pasta custaria a meta de 16 ms. *(assistente)*
- **Sob demanda**, pelo `lucerna-service`: `lucerna-files.socket` + `lucerna-files.service` do systemd do usuário; sai depois de 60 s sem clientes e sem trabalho pendente. Trabalho pendente *(assistente)*: operação em andamento, pausada ou esperando resposta de conflito, e o item de progresso no shell enquanto existir.
- **Concorrência entre processos deixa de existir:** só o serviço abre o diário e a fila. A TUI, a GUI, a CLI e o shell são clientes; dois deles enfileirando ao mesmo tempo é só ordem de chegada numa fila só. O socket é a trava: um segundo `serve` não consegue ouvir o mesmo caminho e sai.

**Cliente sem o serviço rodando** *(assistente; resolve a pergunta da CLI)*: a CLI, a TUI e a GUI **sobem o serviço**, nunca esperam por ele indefinidamente.

1. Conectam ao `files.sock`. Com a unidade do systemd instalada, a própria conexão sobe o serviço (ativação por socket), como em qualquer app do ecossistema.
2. Sem socket (sem systemd, unidade não instalada, desenvolvimento): o cliente roda `lucerna-files serve` solto (novo grupo de sessão, saída para o journal ou um log em `$XDG_STATE_HOME`) e tenta conectar por até 2 s, com espera crescente. Dois clientes fazendo isso juntos: o segundo `serve` falha ao ouvir e sai; os dois clientes conectam no primeiro.
3. Sem conseguir em 2 s: a CLI sai com o código 2 ("sem serviço") e a mensagem do porquê; a TUI e a GUI continuam navegando (as leituras não dependem do serviço) e mostram as ações de mudança como indisponíveis, com a causa.

**Retomada depois de uma queda** *(assistente)*:

- O diário (`~/.local/share/lucerna/files/journal.db`, SQLite em WAL, molde do `clipboard`) grava a operação inteira antes de começar e cada item ao terminar (origem, destino, o que foi para a lixeira, o modo antigo). Cópias usam um temporário no destino (`.<nome>.lucerna-part`) renomeado só no fim.
- O serviço cai: o systemd o reinicia (`Restart=on-failure`); sem systemd, o próximo cliente que conectar o sobe (passo 2 acima), com a mesma retomada. Na subida, toda operação que estava "em andamento" vira **"interrompida"** e fica **pausada**, sem retomar sozinha: os itens já concluídos ficam como estão, os temporários `.lucerna-part` daquela operação são apagados, e o item que estava pela metade recomeça do zero quando o usuário retomar.
- Uma operação interrompida aparece na fila da TUI/GUI e, com o shell presente, na central (pelo item de progresso, com "Retomar" e "Cancelar"). Depois de um reinício do computador, ela aparece na próxima vez que o serviço subir (qualquer cliente, inclusive o shell ao pedir uma busca); nada sobe o serviço no login só por isso.
- "Cancelar" numa interrompida oferece desfazer o que já foi feito, pelo diário.

**Desfazer** *(assistente)*: toda operação concluída fica no diário com o necessário para voltar: o caminho original (mover, renomear), a entrada da lixeira (apagar e **sobrescrever**: o arquivo antigo vai para a lixeira freedesktop antes da sobrescrita, inclusive numa extração por cima), o modo e o dono antigos (permissões em massa). Desfazer é uma operação nova na fila, com os mesmos conflitos.

**Métodos e tópicos** *(assistente; nomes finais no plano)*:

- tópicos: `ops` (a fila: id, tipo, origem e destino resumidos, bytes, itens, velocidade, estado, conflito pendente), `history` (as últimas operações concluídas, desfazíveis), `mounts` (as conexões de rede montadas pelo explorador);
- métodos: `ops.enqueue`, `ops.pause`, `ops.resume`, `ops.cancel`, `ops.resolve {id, escolha, para_todos}`, `ops.undo {id?}`, `search.files {consulta, tipo, limite}` (a mesma que o launcher usa), `frecency.visit`, `frecency.query`, `mounts.connect`, `mounts.disconnect`, `drives.mount`, `drives.unmount`, `drives.eject`.

### Como as peças se falam

```
   ┌────────────────────── lucerna-shell ───────────────────────┐
   │ central (progresso)   launcher (busca)   aviso de pendrive  │
   └───▲──────────────────────────┬─────────────────▲───────────┘
       │ shell.sock               │ files.sock      │ socket do lucernad
       │ progress.set/clear       │ (só enquanto    │ (módulo de discos,
       │                          │  pesquisa)      │  sem o explorador)
   ┌───┴──────────────────────────▼───┐        ┌────┴─────┐
   │ lucerna-files serve (dono único) │        │ lucernad │
   └───▲──────────────▲───────────────┘        └──────────┘
       │ files.sock   │ files.sock
   ┌───┴───────────┐ ┌┴──────────────────┐
   │ lucerna-files │ │ lucerna-files-gui │── portal FileChooser,
   │ TUI + CLI     │ │ um processo,      │   FileManager1 (D-Bus da sessão)
   └───────────────┘ │ várias janelas    │
                     └───────────────────┘
   listagem, prévia e busca da pasta aberta: no processo da vista, pelo núcleo
```

- **Progresso na central sem o shell segurar o serviço:** o shell não fica conectado ao `files.sock` (uma conexão permanente impediria a saída por ociosidade). É o serviço que, enquanto tem operação, põe um **item de progresso** no `shell.sock` (`progress.set {id, título, fração, detalhe, estado, ações}`), do mesmo jeito que um app põe um item na barra: pertence à conexão, some se o serviço sai, é reaplicado quando o shell recarrega. Os botões da central (pausar, retomar, cancelar, resolver conflito) fazem o shell conectar ao `files.sock` e chamar o método; o serviço está de pé, porque tem trabalho. **Sem o shell** (o `shell.sock` não existe ou caiu), o item vira nada, sem erro e sem espera, como a barra no SDK: a operação segue igual, o progresso fica na fila da TUI/GUI e o fim é avisado pela notificação do sistema; o SDK reaplica o item se o shell aparecer depois. *(assistente; pede a mudança no protocolo descrita adiante)*
- **Launcher:** `services/FileSearch.qml` conecta ao `files.sock` quando o launcher abre a busca de arquivos, pede `search.files` e solta ao fechar; o serviço sobe pela ativação e sai 60 s depois. Sem o explorador instalado, o launcher continua no `file-search.sh` (princípio do adaptador). *(assistente)*
- **TUI ↔ GUI:** "abrir na GUI" roda `lucerna-files-gui --open <pasta> [--select <arquivo>]`; com a GUI já aberta, o pedido vai pelo D-Bus ao processo existente, que abre uma janela nova *(assistente)*. "Abrir na TUI" pede `tui.open` ao shell (ou abre o terminal configurado, sem shell). A sessão (abas) é gravada pelo `lucerna-files-app` num formato só.
- **Configuração e atalhos:** `~/.config/lucerna/files.toml`, lido pelas vistas e pelo serviço, observado (mudou, aplica sem reiniciar).

### GUI: processo único *(assistente)*

- `lucerna-files-gui` tem um nome no D-Bus da sessão (`org.lucerna.Files`) e atende ali o `org.freedesktop.FileManager1` e o backend do portal (`org.freedesktop.impl.portal.FileChooser`). A segunda chamada a `lucerna-files-gui` só repassa os argumentos ao processo que já tem o nome e sai.
- Várias janelas no mesmo processo dividem o motor QML, o cache de miniaturas e a conexão com o serviço; por isso "nova janela" fica abaixo de 80 ms.
- Ativado pelo D-Bus (portal e `FileManager1`) quando não está aberto; sem janelas e sem diálogo do portal pendente, sai depois de um tempo curto (padrão 30 s), para o próximo diálogo do portal abrir já quente.

### Tema

- **O shell grava o tema resolvido num JSON público** *(assistente)*: os tokens já calculados, inclusive os do tema gerado do papel de parede, com os grupos do `ThemeManager` (`colors`, `fonts`, `radius`, `spacing`, `motion`) e `dark`, gravado de forma atômica a cada troca. O local e o formato exatos ficam para um spec pequeno do shell (sugestão: `$XDG_STATE_HOME/lucerna/theme.json`, com um campo `schema`).
- A GUI consome pelo módulo de tema do padrão de GUI (abaixo). Sem o arquivo (shell ausente), usa o tema padrão do Lucerna (Catppuccin Mocha), embutido.
- A TUI usa por padrão a paleta do terminal; com `tema = "lucerna"` na configuração, usa as cores do mesmo arquivo em truecolor. Ícones por Nerd Font, com reserva sem ícones.

### Discos no shell *(assistente)*

O aviso de pendrive precisa existir com o explorador fechado, e o serviço do explorador sai por ociosidade. Então quem observa o udisks2 para o shell é o `lucernad` (um módulo de discos, ou uma extensão do `system`, que já lê os discos), **sem depender do explorador**: o shell mostra o aviso e a lista na central, e "Abrir" chama o gerenciador de arquivos padrão pelo `org.freedesktop.FileManager1` (que é o `lucerna-files-gui` quando instalado). "Ejetar" na central é do `lucernad`. No explorador, montar e ejetar vão pelo serviço dele, com o próprio cliente udisks2 (no núcleo), para o explorador funcionar sem o `lucernad`. O desenho do módulo fica para um spec do `lucernad`.

### Integração com o sistema (sem gvfs)

- **Montagens locais:** udisks2 por `zbus` (D-Bus do sistema), no serviço, com prazo em toda chamada. Montar, desmontar, ejetar, desligar a energia, desbloquear LUKS (a senha é pedida pela vista que pediu).
- **Rede e nuvem:** "Conexões" na configuração (SFTP, SMB, rclone). Montar = `rclone mount` / `sshfs` / `mount.cifs` como unidade transitória do systemd do usuário (`systemd-run --user`), para a montagem não depender da janela nem do serviço; sem o systemd do usuário, o serviço roda o mesmo comando solto, num grupo de sessão próprio, e guarda o pid para desmontar depois. *(assistente)*
- **Lixeira:** especificação freedesktop (`~/.local/share/Trash` e `.Trash-<uid>` em cada disco).
- **Miniaturas:** especificação freedesktop (`~/.cache/thumbnails/…`, PNG com `Thumb::URI` e `Thumb::MTime`), compartilhadas com qualquer app e com o shell.
- **Abrir com / padrões:** `mimeapps.list` lido e escrito pela especificação.
- **Recentes e favoritos:** `recently-used.xbel` e os favoritos do GTK (`~/.config/gtk-3.0/bookmarks`, só o formato de texto, sem GTK). *(assistente)*

## Uso sem o Lucerna **(usuário: princípio; detalhes: assistente)**

O explorador se instala e funciona sozinho, em qualquer distribuição e ambiente (KDE, GNOME, Hyprland sem o shell, outro compositor Wayland), com ou sem systemd. O pacote não depende do shell nem do `lucernad`; o `lucerna-sdk` entra só como biblioteca, na compilação. A integração com o shell liga quando ele está presente e some em silêncio quando não está: nada falha, trava ou espera por causa da ausência de uma peça do Lucerna.

| Sem… | O que continua igual | O que muda |
|---|---|---|
| **o shell** (`lucerna-shell`) | Navegação, operações, fila, diário, desfazer, busca, TUI, GUI, CLI, portal e `FileManager1`. | **Tema:** a GUI usa o tema padrão embutido (Catppuccin Mocha); a TUI usa a paleta do terminal, e `tema = "lucerna"` cai no mesmo padrão embutido. **Progresso:** o `progress.set` vira nada, sem erro e sem espera; o progresso fica na fila da TUI/GUI e o fim é avisado pela notificação do sistema (`org.freedesktop.Notifications`, que o KDE e o GNOME atendem). **Abrir na TUI:** abre o terminal configurado, sem o `tui.open`. **Some:** "definir como papel de parede", a busca do launcher e o arrastar para os painéis (são do shell). |
| **o `lucernad`** | Montar, desmontar e ejetar pelo próprio explorador (cliente udisks2 no núcleo); copiar e colar arquivos (a área de transferência é a do sistema). | Sem o aviso de pendrive e sem o histórico da área de transferência, que são do `lucernad`. |
| **o Hyprland** | Tudo: nada no explorador fala com o Hyprland. A GUI é Qt Quick comum no Wayland; o X11 não é testado, mas nada o recusa. | Fora do Hyprland, o `portals.conf` sugerido não se aplica: o KDE e o GNOME já têm o próprio seletor de arquivos, e o do explorador só é usado se o usuário o escolher no `portals.conf`. |
| **o systemd** | O serviço, a fila, a retomada e as montagens. | **Serviço:** sem ativação por socket, o cliente sobe o `lucerna-files serve` sozinho (passo 2 de "Cliente sem o serviço rodando"); depois de uma queda, quem o sobe de novo é o próximo cliente, não o `Restart=on-failure`. **Rede e nuvem:** sem `systemd-run --user`, o serviço roda o `rclone`/`sshfs`/`mount.cifs` solto, num grupo de sessão próprio. |

Outras ausências *(assistente)*:

- **Sem udisks2:** montar, desmontar e ejetar somem com uma explicação; o resto segue.
- **Sem D-Bus de sessão:** cada `lucerna-files-gui` vira um processo próprio (sem instância única, sem portal e sem `FileManager1`).
- **`FileManager1` no KDE e no GNOME:** a GUI pede o nome sem tomá-lo de quem já tem; com o Dolphin ou o Nautilus registrado, ela segue sem ele, em silêncio. O arquivo de ativação do `FileManager1` no D-Bus é opcional no pacote, para não disputar com o gerenciador do ambiente.
- **Testes:** o CI roda a TUI, a CLI e o serviço sem shell, sem `lucernad` e sem systemd (o `shell.sock` falso só entra nos testes da integração).

## Padrão de GUI do ecossistema **(usuário: padronizar; desenho: assistente)**

Vale para todo app do Lucerna que ganhar janela própria. O explorador é o primeiro a segui-lo.

### Estrutura de um app com GUI

```
lucerna-<app>/
└── crates/
    ├── lucerna-<app>-core/      Model: lógica, sem interface (feature-first + core + share)
    ├── lucerna-<app>-app/       ViewModel: estado e ações, sem ratatui nem Qt; testado sem janela
    ├── lucerna-<app>/           binário: TUI + CLI + `serve`
    └── lucerna-<app>-gui/       binário: View Qt
        ├── build.rs             cxx-qt-build: registra o módulo QML `Lucerna.<App>`
        ├── src/
        │   ├── main.rs          três linhas: lucerna_gui::run(App)
        │   └── <feature>/       uma pasta por recurso, com o QObject da ponte (ex.: `listing/model.rs`)
        └── qml/
            ├── Main.qml
            └── <feature>/       as vistas do recurso (ex.: `listing/ListingView.qml`)
```

- O mesmo recorte por recurso nas três camadas: `core/src/listing/`, `app/src/listing/`, `gui/src/listing/` e `gui/qml/listing/`. Quem procura um recurso acha as quatro pastas com o mesmo nome.
- **Model** = núcleo + serviço (pelo `lucerna-ipc`). **ViewModel** = o crate `-app`, em Rust puro: estado observável e comandos, eventos de mudança por canal. **View** = QML, só apresentação.
- **A ponte cxx-qt** é fina: um `QObject` por ViewModel, com `Q_PROPERTY` para o estado, `Q_INVOKABLE` para os comandos e sinais para os eventos. Listas grandes são `QAbstractListModel` alimentados por **diferenças em lotes** (inserir, remover, mudar intervalos), nunca a lista inteira. Nenhuma regra de negócio na ponte.
- **Assíncrono:** o runtime tokio roda em threads próprias; o resultado volta para a thread do Qt pela fila do cxx-qt (`CxxQtThread::queue`), agrupado por quadro. A thread do Qt nunca espera I/O.
- **QML sem lógica:** nada de estado em JS além do visual (hover, animação); toda decisão vem do ViewModel. As ações vêm do catálogo do `-app`, então o menu de contexto, a paleta e os atalhos da GUI são gerados da mesma tabela que a TUI usa.
- **Um processo, várias janelas, nome único no D-Bus** (`org.lucerna.<App>`), como o explorador.

### O que é comum e onde mora *(assistente: no `lucerna-sdk`)*

- **Crate `lucerna-gui`** (Rust + cxx-qt):
  - o arranque: `QGuiApplication`, motor QML com cache AOT, instância única pelo D-Bus com repasse de argumentos, várias janelas, saída por ociosidade opcional;
  - a ponte assíncrona tokio ↔ Qt e um modelo de lista genérico com diferenças em lotes;
  - o cliente do serviço do app (sobre o `lucerna-ipc`), com a subida do serviço descrita em "Cliente sem o serviço rodando";
  - o leitor do tema resolvido, com observação do arquivo e o tema padrão embutido.
- **Módulo QML `Lucerna.Ui`**, embutido no binário de cada app pelo crate (sem instalação no sistema, sem versão desencontrada):
  - `Theme` (singleton): os mesmos grupos e nomes de tokens do `ThemeManager` do shell (`Theme.colors.accent`, `Theme.spacing.normal`, `Theme.motion.spatial`…), ao vivo;
  - componentes base, sobre os templates do QtQuick.Controls (acessibilidade do Qt de graça): `AppWindow`, `Button`, `IconButton`, `TextField`, `SearchField`, `Switch`, `CheckBox`, `ComboBox`, `Menu`/`ContextMenu`, `Dialog`, `ToolTip`, `ScrollBar`, `ListRow`, `Sidebar`, `TabBar`, `SplitView`, `CommandPalette`, `ProgressRow`, `EmptyState`, `Icon` (Material Symbols, como o shell e os manifestos);
  - regras: foco visível, navegação por teclado, contraste conferido no CI pelos temas embutidos, "reduzir movimento" pela escala de movimento do tema.

**Por que no `lucerna-sdk`:** o SDK já é "o molde que todos os apps copiam" (protocolo, esqueleto do serviço e da linha de comando) e já tem, na tabela de versões, a API de componentes; pôr a GUI ali mantém um lugar só para o que é comum e uma versão só por release. O custo é o CI do SDK ganhar Qt, contido num job próprio e num crate fora dos `default-members`. **Alternativa registrada:** um repositório próprio (`lucerna-ui`), que valeria se o shell um dia passar a usar o mesmo `Lucerna.Ui` nos painéis dos apps (o host de painéis); hoje os componentes do shell dependem do Quickshell, então essa unificação é futuro, não objetivo.

**Ordem:** o `lucerna-gui` e o `Lucerna.Ui` nascem no SDK com o que a v2 do explorador precisa (e um `lucerna-hello-gui` mínimo como exemplo); o que for só do explorador fica no repositório dele. O explorador usa o SDK por revisão do git, como o `lucernad`, até a publicação.

### O que muda no `lucerna-sdk` (PR separado lá)

- **`docs/ecossistema.md`:**
  - "Repositórios": linha do `lucerna-files`.
  - "Estrutura de cada app": a quinta parte opcional, **GUI** (Qt Quick + cxx-qt, cliente do serviço como a TUI e o painel), e o crate de ViewModel (`-app`) comum à TUI e à GUI; o recorte por recurso nas camadas.
  - "Protocolo"/binários: com GUI, dois binários (`lucerna-<app>` e `lucerna-<app>-gui`).
  - "O que o SDK oferece": a base de GUI (`lucerna-gui`, `Lucerna.Ui`) e o item de progresso no shell.
  - "Serviço sob demanda": o cliente sobe o serviço sozinho quando não há systemd (o passo 2 de "Cliente sem o serviço rodando").
  - Uma seção "Tema" apontando o contrato do tema resolvido (spec do shell).
- **Spec do protocolo / `lucerna-protocol` / `lucerna-service`:** `progress.set` e `progress.clear` no `shell.sock` (protocolo `1.1`, um `MINOR`: novo e opcional), pertencentes à conexão, reaplicados na reconexão e contados como trabalho pendente, como a barra; e a subida do serviço pelo cliente no `lucerna-ipc`.
- **Crates novos:** `lucerna-gui`, o módulo `Lucerna.Ui` e o app `lucerna-hello-gui`; job `gui` no CI com Qt 6.

Fora do SDK, em PRs próprios: no `lucerna-shell`, o spec do tema resolvido, o `progress.*` na central e os adaptadores `FileSearch.qml`/`FileOps.qml`; no `lucernad`, o módulo de discos e a nota no item 6 da proposta (a busca de arquivos do launcher fica com o explorador).

## Recursos

Origem: **U** = decisão do usuário (inspirações e integração); **P** = proposta do assistente, incluída; **N** = proposta nova deste documento (assistente), incluída para aprovação.

### Navegação e visão

- Abas, painel dividido e **painel duplo de verdade** (F5 copia para o outro painel, F6 move) — U.
- Visões: lista com colunas, ícones (GUI), **colunas Miller** (as duas) — U.
- Atalhos amigáveis por padrão; **modo Vim opcional** — U. Tudo também pela **paleta de comandos** — P.
- **Pular por frecência** (estilo zoxide) — P.
- **Filtro ao vivo** na pasta e **selecionar por padrão** (`*.jpg`) — N.
- **Restaurar sessão** (abas, painéis, pastas, seleção) — P.
- Barra lateral: favoritos, discos, conexões, lixeira, pesquisas salvas.
- Ordem natural; pastas primeiro; ocultos alternáveis; arquivo `.hidden` respeitado — N.
- Atualização ao vivo quando algo muda fora (inotify).

### Pré-visualização

- Painel de prévia: texto com realce (carregado sob demanda), imagens, pastas, compactados (lista), metadados de mídia, PDF e vídeo por miniatura — U.
- **Imagens na TUI** por kitty/sixel com blocos como reserva — U.
- **Visualização rápida com espaço** (quick look), na GUI no estilo dos painéis do shell — P.

### Operações

- **Fila** com pausa, retomada, cancelamento, prioridade e conflitos (sobrescrever, pular, renomear, manter o mais novo, "para todos"); sobrevive à janela fechada pelo serviço — U.
- **Diário e desfazer para toda operação**, inclusive sobrescritas (o antigo vai para a lixeira) e permissões em massa (o modo antigo no diário); retomada depois de queda — P.
- Cópia segura: temporário no destino, `fsync`, renomear; mover entre discos = copiar, conferir, só então apagar; preserva permissões, datas, xattrs e esparsos; **reflink** quando possível; checksum opcional — N.
- **Lixeira freedesktop**; apagar definitivo só com confirmação explícita — P.
- **Renomeação em massa** no nível do Thunar: buscar e substituir (com expressão regular), numeração, datas (do arquivo e EXIF), maiúsculas/minúsculas, inserir/remover, extensão; **prévia**, detecção de colisão, desfazer — U. Na TUI, também "editar nomes no `$EDITOR`" (estilo `vidir`) — N.
- **Compactados navegáveis como pastas**; compactar e extrair pela fila (zip, tar, gz, xz, zst, 7z por crates Rust; rar só leitura com `unrar` se existir) — P.
- **Checksums** (SHA-256, SHA-1, MD5, BLAKE3) e conferência contra `.sha256` — P.
- **Editor de permissões** (modo, dono, grupo, recursivo com prévia) — P.

### Busca

- **Busca por nome** assíncrona, no núcleo, `.gitignore` opcional, cancelável a cada tecla — U.
- **Busca por conteúdo** (estilo ripgrep) com resultados ao vivo — P.
- **Pesquisas salvas** e **etiquetas/cores por xattr** (`user.xdg.tags`, com reserva em SQLite) — P.
- Filtros por tipo, tamanho, data, etiqueta; os mesmos tipos do launcher — N.

### Menus e personalização

- **Menu de contexto rico** e **propriedades detalhadas** (tamanho real e ocupado, contagem recursiva assíncrona, datas, MIME, link, montagem, permissões) — U.
- **Ações personalizadas** (scripts) no menu, como as do Thunar: `%f`/`%F`/`%d`, condição por tipo e padrão, opção de rodar na fila — P.
- **Editor de "abrir com"/apps padrão** — P.
- Personalização por pasta gravada no cache, sem arquivos `.directory` espalhados — N.

### Ferramentas

- **Status do git** nas colunas e ramo na barra de caminho, em segundo plano — P.
- **Uso de disco** (estilo ncdu) com **mapa** (treemap na GUI, barras na TUI) — P.
- **Duplicados** e **comparação de pastas** (sincronizar pela fila) — P.
- **Terminal**: "abrir terminal aqui" sincronizado com a pasta; terminal embutido na GUI (v4) — U.

### Integração com o shell e o sistema

- **Tema do Lucerna** ao vivo — U.
- **Progresso na central** mesmo com o explorador fechado, com pausar, cancelar, retomar e responder conflito pela central; notificação ao terminar — U.
- **Launcher busca pelo mesmo núcleo**, pelo serviço — U.
- **Pendrives e discos no shell** e abertura no explorador — U.
- **Arquivos copiados no histórico** (já coberto pelo módulo `clipboard` do `lucernad`: o explorador oferece `text/uri-list` e `x-special/gnome-copied-files`) — U.
- **Seletor de arquivos do sistema** (backend FileChooser do xdg-desktop-portal) — P.
- **`org.freedesktop.FileManager1`**: o "mostrar na pasta" dos outros apps abre o explorador — N.
- **Arrastar e soltar** entre o explorador, os outros apps e os painéis do shell — P.
- **Miniaturas compartilhadas** com o shell (seletor de papel de parede, launcher) — P.
- **"Definir como papel de parede"** no menu de imagens e vídeos, pelo IPC do shell; sem o shell, a ação não aparece — N.
- **Abrir a pasta da TUI na GUI e vice-versa**; configuração e atalhos compartilhados — P.

### Qualidade

- **pt-BR** em tudo (catálogo de tradução para o futuro) e **acessibilidade** (teclado nas duas vistas; na GUI, papéis e nomes acessíveis, foco visível, contraste, reduzir movimento) — P.
- **CLI** para scripts: `lucerna-files cp|mv|trash|undo|ops|search|mount`, pela fila e com `--json` — N.

### Futuro registrado (N)

- Celular pelo `lucerna-phone` como mais uma montagem, e "enviar para o celular".
- Operações como administrador por um auxiliar pequeno via polkit (só operações específicas, nunca a janela como root).
- Índice opcional de nomes em SQLite com trigram (molde da busca do `clipboard`).
- O shell usar o mesmo `Lucerna.Ui` nos painéis dos apps (unificação dos componentes).

## Recursos por fase

**v1: núcleo + serviço + TUI** (ordem: usuário)

- `lucerna-files-core`, `lucerna-files-app` e o serviço (`serve`) com fila, diário, retomada, conflitos, desfazer, cópia segura, lixeira, renomeação em massa (prévia e `$EDITOR`), compactados, checksums, permissões, montagens udisks2, frecência, busca.
- TUI: abas, painel duplo, Miller, filtro, paleta, frecência, modo Vim opcional, prévia com imagens, busca por nome e conteúdo, sessão, propriedades.
- Abrir com/mimeapps, ações personalizadas, favoritos, configuração TOML, CLI, pt-BR; unidades do systemd e subida sem systemd.
- Repositório, CI (`fmt`, `clippy`, `test`, `deps`), `dev/bench.sh` com as metas da TUI, do serviço e da listagem.

**v2: GUI (e o padrão de GUI)**

- No `lucerna-sdk`: `lucerna-gui`, `Lucerna.Ui` e `lucerna-hello-gui`, com o que o explorador precisa.
- `lucerna-files-gui` sobre o mesmo `lucerna-files-app`: visões (ícones, lista, Miller), painel duplo, abas, menus ricos, propriedades, quick look, miniaturas, arrastar e soltar, renomeação em massa, paleta; processo único com várias janelas; `FileManager1`; "abrir terminal aqui"; acessibilidade.
- Tema pelo arquivo do shell (com o padrão embutido como reserva).
- CI com o job `gui`; `dev/bench.sh` mede abertura, nova janela e RSS da GUI.

**v3: shell e portal**

- `lucerna-sdk`: `progress.*` no `shell.sock` (protocolo `1.1`).
- `lucerna-shell`: o tema resolvido em JSON público; o progresso na central; `FileSearch.qml` pelo `files.sock`; arrastar e soltar com os painéis; "definir como papel de parede"; miniaturas compartilhadas.
- `lucernad`: o módulo de discos (aviso de pendrive, ejetar), sem depender do explorador.
- Backend do portal FileChooser e o `portals.conf` sugerido para o Hyprland.

**v4: extras, por prioridade** *(assistente)*

1. Etiquetas por xattr e pesquisas salvas.
2. Status do git.
3. Conexões de rede e nuvem (rclone, sshfs, SMB).
4. Uso de disco com mapa.
5. Duplicados e comparação de pastas.
6. Terminal embutido na GUI (emulador em Rust, ex. `alacritty_terminal`, desenhado num item do Qt Quick; só se mantiver as metas de abertura e RSS, carregado sob demanda).
7. Índice opcional de nomes; `lucerna-phone`; auxiliar de administrador.

Cada fase é um ou mais planos próprios, com as metas de desempenho conferidas no fim.

## Convenções do repositório

Iguais às do ecossistema: GPL-3.0-only; `main` protegida por ruleset (PR obrigatório, conversas resolvidas, só merge commit, sem deleção nem force push); commits em português; README com "Situação" e "Ligação com o Lucerna"; `docs/proposta.md` como visão geral; templates de issue e de PR; checks obrigatórios (`fmt`, `clippy`, `test`, `deps`, `gui`) no ruleset quando o CI existir; linha nova no perfil da org e no `ecossistema.md`.

## O que fica para o construtor

- Escolher os crates: inotify (`notify` ou direto pelo `rustix`), busca (`ignore` + `grep-searcher`), realce (`syntect` enxuto), compactados (`zip`, `tar`, `zstd`, `sevenz-rust`), git (`gix` com features mínimas ou o `git` externo), área de transferência do Wayland (`wl-clipboard-rs`), terminal embutido (v4).
- Fechar os nomes e campos dos métodos e tópicos do serviço e do `progress.set`.
- O formato do diário e a retenção do desfazer (quantas operações, por quanto tempo); o que fazer com a lixeira cheia por sobrescritas antigas.
- O contrato do tema resolvido, com o spec do shell.
- O tempo de ociosidade da GUI e do serviço, se o padrão de 60 s não servir.
- Os atalhos padrão (amigáveis) e o conjunto Vim, numa tabela única no `lucerna-files-app`.
- A máquina e as pastas de referência do `dev/bench.sh` (1 mil, 100 mil, a pasta pessoal).

# Proposta do lucerna-files

O `lucerna-files` é um explorador de arquivos em Rust. Funciona sozinho, com uma TUI e uma GUI em Qt, e se integra ao Lucerna.

Segue a estrutura de apps do [ecossistema](https://github.com/lucerna-desktop/lucerna-sdk/blob/main/docs/ecossistema.md): núcleo, serviço, TUI e GUI. É o primeiro app com GUI própria, e ela vira o padrão para os outros.

O desenho completo está em [`superpowers/specs/2026-10-27-explorador-design.md`](superpowers/specs/2026-10-27-explorador-design.md).

## A ideia

Pegar o melhor de cada explorador, sem o peso de nenhum:

- do Thunar e do PCManFM, abrir na hora com pouca memória e a renomeação em massa;
- do Nautilus, ser simples à primeira vista, com o poder atrás de atalhos;
- do Dolphin, as abas, o painel dividido, a pré-visualização e a nuvem por montagens;
- do Nemo, o menu de contexto rico e as propriedades detalhadas;
- do Krusader e do Double Commander, o painel duplo e a fila de operações com pausa;
- do ranger, os atalhos estilo Vim (opcionais), as colunas e as imagens no terminal.

Nada do KDE, do GNOME ou do gvfs.

## Serviço

O serviço é o único que mexe nos arquivos: copiar, mover, renomear, mandar para a lixeira. Ele guarda a fila e um diário de tudo o que fez, e por isso:

- uma cópia grande continua com a janela fechada;
- toda operação se desfaz, até uma sobrescrita;
- depois de uma queda, a operação volta pausada, pronta para retomar.

Ele sobe quando alguém precisa e sai sozinho depois de um tempo parado. A TUI, a GUI, a linha de comando e o shell são clientes dele.

## TUI e GUI

As duas fazem o mesmo, com os mesmos atalhos e a mesma configuração. A TUI roda em qualquer terminal, até por SSH. A GUI é uma janela leve em Qt Quick.

- abas, painel duplo e colunas;
- pré-visualização, inclusive de imagens no terminal, e a visualização rápida com espaço;
- busca por nome e por conteúdo;
- paleta de comandos e o pulo para as pastas mais usadas;
- compactados abertos como pastas;
- lixeira, renomeação em massa com prévia, permissões, checksums.

## Sozinho ou no Lucerna

O explorador se instala e funciona sozinho, em qualquer distribuição e ambiente (KDE, GNOME e outros), com ou sem systemd, sem o shell e sem o `lucernad`. O que vem abaixo é um extra, que liga quando o shell está presente:


- usa o tema do shell, inclusive o gerado do papel de parede;
- o progresso das operações aparece na central, mesmo com o explorador fechado;
- o launcher busca arquivos pelo mesmo núcleo;
- pendrives aparecem no shell e abrem no explorador;
- arquivos copiados entram no histórico da área de transferência;
- vira o seletor de arquivos de todos os apps (o "Abrir" e o "Salvar").

## Ordem

1. Núcleo, serviço e TUI.
2. GUI.
3. Integração com o shell e o seletor de arquivos.
4. Os extras: etiquetas, git, nuvem, uso de disco, duplicados, terminal embutido.

## Armazenamento

As pastas seguem as regras de "Pastas e dados" no [ecossistema](https://github.com/lucerna-desktop/lucerna-sdk/blob/main/docs/ecossistema.md): a configuração em `~/.config/lucerna/files.toml`, o diário em `~/.local/share/lucerna/files/` e o cache em `~/.cache/lucerna/files/`. A lixeira e as miniaturas são as do padrão freedesktop, as mesmas dos outros apps.

## Em aberto

- O formato do tema que o shell publica, num spec do shell.
- O item de progresso no protocolo, num PR do `lucerna-sdk`.

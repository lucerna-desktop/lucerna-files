# Design da GUI

Protótipos da GUI do `lucerna-files` (v2 no [spec](../superpowers/specs/2026-10-27-explorador-design.md)). Substituem o arquivo do Figma "Lucerna Files — GUI".

- [`tela-principal.html`](tela-principal.html): a tela principal em HTML e CSS, com ícones do [Lucide](https://lucide.dev). Abra direto no navegador.
- [`rascunho-v3.png`](rascunho-v3.png): o rascunho v3 que serviu de base, com as cores antigas.

## Tokens

Os nomes seguem o tema do `lucerna-shell` (`themes/*.json`), para a GUI em QML usar os mesmos:

- `--colors-*`: as 13 cores do tema (`base`, `surface`, `raised`, `track`, `border`, `accent`, `accentText`, `text`, `textMuted`, `textFaint`, `warning`, `danger`, `success`).
- `--derived-*`: misturas calculadas a partir das cores do tema; não existem no shell, e a GUI calcula o mesmo em QML.
- `--spacing-*` (`tiny` 4, `small` 8, `normal` 12, `large` 20), `--radius-*` (`small` 8, `normal` 14, `large` 22) e `--font-size-*` (`small` 11, `normal` 13, `large` 16, `huge` 44).
- Fontes: Rubik no texto e JetBrains Mono no mono (no shell, a JetBrainsMonoNL Nerd Font Mono).

## Temas

Os 12 temas do shell estão em blocos `[data-theme]`; o padrão é Catppuccin Mocha. Para trocar:

- o seletor no canto inferior direito, fora da janela;
- `?tema=<nome>` na URL (ex.: `?tema=nord`, `?tema=gruvbox`);
- `?grade=1` mostra os 12 lado a lado.

## Ajustes em relação ao rascunho v3

- Pastas num tom apagado do accent; o accent forte fica para seleção, foco, aba ativa e progresso.
- Painel de detalhes sem o vazio: "Abrir com…" como ação principal logo abaixo das etiquetas e "Definir como papel de parede" como ação secundária.
- Rótulos em `textFaint` e valores em `text` nos metadados.
- Ícones de arquivo com a cor do tipo (PDF, JSON, Markdown).
- Estados de hover e foco, com navegação por teclado na grade e na barra lateral.
- Busca com mais espaço na toolbar e botão do painel lateral num grupo próprio.
- Sem o selo "Futuro · v4", que era nota de design.

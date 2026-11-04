# lucerna-files — Pendências

O que falta fazer no `lucerna-files`, na ordem em que vamos seguir. Cada item vira um PR; ao terminar, ele é marcado aqui.

Cada repositório do ecossistema tem o seu arquivo de pendências na raiz. As decisões ainda em aberto ficam em [`docs/proposta.md`](docs/proposta.md).

## Em ordem

1. [ ] **Aprovar o desenho.** O [spec do explorador](docs/superpowers/specs/2026-10-27-explorador-design.md) está como "desenho para aprovação"; depois dele vem o plano de implementação.
2. [ ] **v1: núcleo, serviço e TUI**, como em "Recursos por fase" do spec: `lucerna-files-core`, `lucerna-files-app`, o serviço (`serve`), a TUI, a CLI, o CI (`fmt`, `clippy`, `test`, `deps`) e o `dev/bench.sh`.
3. [ ] **v2: GUI**, sobre o mesmo `lucerna-files-app`, com o tema pelo arquivo do shell e o job `gui` no CI. Depende do padrão de GUI no `lucerna-sdk` (`lucerna-gui`, `Lucerna.Ui`, `lucerna-hello-gui`), nas Pendências de lá.
4. [ ] **v3: shell e portal.** Aqui: o backend do portal FileChooser e o `portals.conf` sugerido para o Hyprland. O resto da v3 está nas Pendências de cada repositório: o `progress.*` no `lucerna-sdk`, o tema público, o progresso na central e o `FileSearch.qml` no `lucerna-shell`, o módulo de discos no `lucernad`.
5. [ ] **v4: extras**, na ordem de prioridade do spec (etiquetas e pesquisas salvas, status do git, conexões de rede e nuvem, uso de disco, duplicados, terminal embutido, índice de nomes, `lucerna-phone`, auxiliar de administrador).

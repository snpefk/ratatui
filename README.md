# Ratatui

<img align="left" src="https://avatars.githubusercontent.com/u/125200832?s=128&v=4">

`ratatui` is a [Rust](https://www.rust-lang.org) library to build rich terminal user interfaces and
dashboards. It is a community fork of the original [tui-rs](https://github.com/fdehau/tui-rs)
project.

[![Crates.io](https://img.shields.io/crates/v/ratatui?logo=rust&style=for-the-badge)](https://crates.io/crates/ratatui)
[![License](https://img.shields.io/crates/l/ratatui?style=for-the-badge)](./LICENSE)
[![GitHub CI Status](https://img.shields.io/github/actions/workflow/status/tui-rs-revival/ratatui/ci.yml?style=for-the-badge&logo=github)](https://github.com/tui-rs-revival/ratatui/actions?query=workflow%3ACI+)
[![Docs.rs](https://img.shields.io/docsrs/ratatui?logo=rust&style=for-the-badge)](https://docs.rs/crate/ratatui/)
[![Dependency Status](https://deps.rs/repo/github/tui-rs-revival/ratatui/status.svg?style=for-the-badge)](https://deps.rs/repo/github/tui-rs-revival/ratatui)
[![Codecov](https://img.shields.io/codecov/c/github/tui-rs-revival/ratatui?logo=codecov&style=for-the-badge&token=BAQ8SOKEST)](https://app.codecov.io/gh/tui-rs-revival/ratatui)
[![Discord](https://img.shields.io/discord/1070692720437383208?label=discord&logo=discord&style=for-the-badge)](https://discord.gg/pMCEU9hNEj)

![Demo of ratatui](./assets/demo.gif)

<details>
<summary>Table of Contents</summary>

* [Ratatui](#ratatui)
  * [Installation](#installation)
  * [Introduction](#introduction)
  * [Quickstart](#quickstart)
  * [Status of this fork](#status-of-this-fork)
  * [Rust version requirements](#rust-version-requirements)
  * [Documentation](#documentation)
  * [Examples](#examples)
  * [Widgets](#widgets)
    * [Built in](#built-in)
    * [Third\-party libraries, bootstrapping templates and widgets](#third-party-libraries-bootstrapping-templates-and-widgets)
  * [Apps](#apps)
  * [Alternatives](#alternatives)
  * [Acknowledgements](#acknowledgements)
  * [License](#license)

</details>

## Installation

```shell
cargo add ratatui --features all-widgets
```

Or modify your `Cargo.toml`

```toml
[dependencies]
ratatui = { version = "0.21.0", features = ["all-widgets"]}
```

Ratatui is mostly backwards compatible with `tui-rs`. To migrate an existing project, it may be
easier to rename the ratatui dependency to `tui` rather than updating every usage of the crate.
E.g.:

```toml
[dependencies]
tui = { package = "ratatui", version = "0.21.0", features = ["all-widgets"]}
```

## Introduction

`ratatui` is a terminal UI library that supports multiple backends:

* [crossterm](https://github.com/crossterm-rs/crossterm) [default]
* [termion](https://github.com/ticki/termion)
* [termwiz](https://github.com/wez/wezterm/tree/master/termwiz)

The library is based on the principle of immediate rendering with intermediate buffers. This means
that at each new frame you should build all widgets that are supposed to be part of the UI. While
providing a great flexibility for rich and interactive UI, this may introduce overhead for highly
dynamic content. So, the implementation try to minimize the number of ansi escapes sequences
generated to draw the updated UI. In practice, given the speed of `Rust` the overhead rather comes
from the terminal emulator than the library itself.

Moreover, the library does not provide any input handling nor any event system and
you may rely on the previously cited libraries to achieve such features.

## Quickstart

The following example demonstrates the minimal amount of code necessary to setup a terminal and
render "Hello World!". The full code for this example which contains a little more detail is in
[hello_world.rs](./examples/hello_world.rs). For more guidance on how to create Ratatui apps, see
the [Docs](https://docs.rs/ratatui) and [Examples](#examples). There is also a starter template
available at [rust-tui-template](https://github.com/tui-rs-revival/rust-tui-template).

```rust
fn main() -> Result<(), Box<dyn Error>> {
    let mut terminal = setup_terminal()?;
    run(&mut terminal)?;
    restore_terminal(&mut terminal)?;
    Ok(())
}

fn setup_terminal() -> Result<Terminal<CrosstermBackend<Stdout>>, Box<dyn Error>> {
    let mut stdout = io::stdout();
    enable_raw_mode()?;
    execute!(stdout, EnterAlternateScreen)?;
    Ok(Terminal::new(CrosstermBackend::new(stdout))?)
}

fn restore_terminal(
    terminal: &mut Terminal<CrosstermBackend<Stdout>>,
) -> Result<(), Box<dyn Error>> {
    disable_raw_mode()?;
    execute!(terminal.backend_mut(), LeaveAlternateScreen,)?;
    Ok(terminal.show_cursor()?)
}

fn run(terminal: &mut Terminal<CrosstermBackend<Stdout>>) -> Result<(), Box<dyn Error>> {
    Ok(loop {
        terminal.draw(|frame| {
            let greeting = Paragraph::new("Hello World!");
            frame.render_widget(greeting, frame.size());
        })?;
        if event::poll(Duration::from_millis(250))? {
            if let Event::Key(key) = event::read()? {
                if KeyCode::Char('q') == key.code {
                    break;
                }
            }
        }
    })
}
```

## Status of this fork

In response to the original maintainer [**Florian Dehau**](https://github.com/fdehau)'s issue
regarding the [future of `tui-rs`](https://github.com/fdehau/tui-rs/issues/654), several members of
the community forked the project and created this crate. We look forward to continuing the work
started by Florian 🚀

In order to organize ourselves, we currently use a [Discord server](https://discord.gg/pMCEU9hNEj),
feel free to join and come chat! There are also plans to implement a [Matrix](https://matrix.org/)
bridge in the near future. **Discord is not a MUST to contribute**. We follow a pretty standard
github centered open source workflow keeping the most important conversations on GitHub, open an
issue or PR and it will be addressed. 😄

Please make sure you read the updated [contributing](./CONTRIBUTING.md) guidelines, especially if
you are interested in working on a PR or issue opened in the previous repository.

## Rust version requirements

Since version 0.21.0, The Minimum Supported Rust Version (MSRV) of `ratatui` is 1.65.0.

## Documentation

The documentation can be found on [docs.rs.](https://docs.rs/ratatui)

## Examples

The demo shown in the gif above is available on all available backends.

```shell
# crossterm
cargo run --example demo
# termion
cargo run --example demo --no-default-features --features=termion
# termwiz
cargo run --example demo --no-default-features --features=termwiz
```

The UI code for the is in [examples/demo/ui.rs](./examples/demo/ui.rs) while the application state is in
[examples/demo/app.rs](./examples/demo/app.rs).

If the user interface contains glyphs that are not displayed correctly by your terminal, you may
want to run the demo without those symbols:

```shell
cargo run --example demo --release -- --tick-rate 200 --enhanced-graphics false
```

More examples are available in the [examples](./examples/) folder.

## Widgets

### Built in

The library comes with the following
[widgets](https://docs.rs/ratatui/latest/ratatui/widgets/index.html):

* [Canvas](https://docs.rs/ratatui/latest/ratatui/widgets/canvas/struct.Canvas.html) which allows
  rendering [points, lines, shapes and a world map](https://docs.rs/ratatui/latest/ratatui/widgets/canvas/index.html)
* [BarChart](https://docs.rs/ratatui/latest/ratatui/widgets/struct.BarChart.html)
* [Block](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Block.html)
* [Calendar](https://docs.rs/ratatui/latest/ratatui/widgets/calendar/index.html)
* [Chart](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Chart.html)
* [Gauge](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Gauge.html)
* [List](https://docs.rs/ratatui/latest/ratatui/widgets/struct.List.html)
* [Paragraph](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Paragraph.html)
* [Sparkline](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Sparkline.html)
* [Table](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Table.html)
* [Tabs](https://docs.rs/ratatui/latest/ratatui/widgets/struct.Tabs.html)

Each wiget has an associated example which can be found in the [examples](./examples/) folder. Run
each examples with cargo (e.g. to run the gauge example `cargo run --example gauge`), and quit by
pressing `q`.

You can also run all examples by running `cargo make run-examples` (requires `cargo-make` that can
be installed with `cargo install cargo-make`).

### Third-party libraries, bootstrapping templates and widgets

* [ansi-to-tui](https://github.com/uttarayan21/ansi-to-tui) — Convert ansi colored text to `tui::text::Text`
* [color-to-tui](https://github.com/uttarayan21/color-to-tui) — Parse hex colors to `tui::style::Color`
* [rust-tui-template](https://github.com/orhun/rust-tui-template) — A template for bootstrapping a Rust TUI application with Tui-rs & crossterm
* [simple-tui-rs](https://github.com/pmsanford/simple-tui-rs) — A simple example tui-rs app
* [tui-builder](https://github.com/jkelleyrtp/tui-builder) — Batteries-included MVC framework for Tui-rs + Crossterm apps
* [tui-clap](https://github.com/kegesch/tui-clap-rs) — Use clap-rs together with Tui-rs
* [tui-log](https://github.com/kegesch/tui-log-rs) — Example of how to use logging with Tui-rs
* [tui-logger](https://github.com/gin66/tui-logger) — Logger and Widget for Tui-rs
* [tui-realm](https://github.com/veeso/tui-realm) — Tui-rs framework to build stateful applications with a React/Elm inspired approach
* [tui-realm-treeview](https://github.com/veeso/tui-realm-treeview) — Treeview component for Tui-realm
* [tui tree widget](https://github.com/EdJoPaTo/tui-rs-tree-widget) — Tree Widget for Tui-rs
* [tui-windows](https://github.com/markatk/tui-windows-rs) — Tui-rs abstraction to handle multiple windows and their rendering
* [tui-textarea](https://github.com/rhysd/tui-textarea): Simple yet powerful multi-line text editor widget supporting several key shortcuts, undo/redo, text search, etc.
* [tui-rs-tree-widgets](https://github.com/EdJoPaTo/tui-rs-tree-widget): Widget for tree data structures.
* [tui-input](https://github.com/sayanarijit/tui-input): TUI input library supporting multiple backends and tui-rs.

## Apps

Check out the list of [close to 40 apps](./APPS.md) using `ratatui`!

## Alternatives

You might want to checkout [Cursive](https://github.com/gyscos/Cursive) for an alternative solution
to build text user interfaces in Rust.

## Acknowledgements

Special thanks to [**Pavel Fomchenkov**](https://github.com/nawok) for his work in designing **an awesome logo** for the ratatui project and tui-rs-revival organization.

## License

[MIT](./LICENSE)

# Cargo — шпаргалка

## Что это и зачем

Cargo — менеджер проектов и сборки Rust. Он создаёт пакет, разрешает зависимости, собирает цели, запускает тесты и формирует документацию. Устанавливается вместе с Rust через rustup. Каталог сторонних пакетов — [crates.io](https://crates.io/).

Проверьте установку через `cargo --version`. Для форматирования и lint установите компоненты `rustfmt` и `clippy` через `rustup component add rustfmt clippy`, если их нет.

## Быстрый старт

```bash
cargo new hello-rust
cd hello-rust
cargo run
```

Cargo создаёт `Cargo.toml`, каталог `src/` и по умолчанию инициализирует Git для нового проекта вне существующего репозитория. Результаты сборки появляются в `target/`.

## Основные команды

| Команда | Назначение | Пример |
|---|---|---|
| `new` | Создаёт бинарный пакет | `cargo new app` |
| `init` | Инициализирует пакет в существующем каталоге | `cargo init` |
| `build` | Собирает debug-версию | `cargo build` |
| `build --release` | Собирает оптимизированную версию | `cargo build --release` |
| `run` | Собирает и запускает бинарник | `cargo run -- arg1` |
| `check` | Проверяет код без полной сборки | `cargo check` |
| `test` | Собирает и запускает тесты | `cargo test` |
| `bench` | Запускает бенчмарки | `cargo bench` |
| `doc` | Генерирует документацию | `cargo doc --open` |
| `fmt` | Форматирует код | `cargo fmt --all` |
| `clippy` | Запускает статический анализатор | `cargo clippy --all-targets --all-features` |
| `clean` | Удаляет `target/` | `cargo clean` |
| `tree` | Показывает дерево зависимостей | `cargo tree` |
| `metadata` | Выводит метаданные проекта | `cargo metadata --no-deps --format-version 1` |
| `update` | Обновляет зависимости в допустимых версиях | `cargo update` |
| `publish --dry-run` | Проверяет пакет перед публикацией | `cargo publish --dry-run` |

## Зависимости

```bash
cargo add serde
cargo add --dev pretty_assertions
cargo add tokio --features full
cargo remove serde
```

Команды `cargo add` и `cargo remove` поставляются с современным Cargo. Если их нет, изменяйте `Cargo.toml` вручную или установите `cargo-edit`. Прямые зависимости добавляются в `[dependencies]`, тестовые и бенчмарковые — в `[dev-dependencies]`.

```toml
[package]
name = "hello-rust"
version = "0.1.0"
edition = "2024"

[dependencies]
serde = "1"

[dev-dependencies]
pretty_assertions = "1"
```

Файл `Cargo.lock` фиксирует конкретные версии. Для бинарного приложения обычно храните его в Git. Для библиотеки решение зависит от политики проекта.

## Профили и цели

```bash
cargo build --release
cargo run --release
cargo build --target x86_64-unknown-linux-gnu
rustup target list --installed
```

Настройки профиля находятся в `Cargo.toml`.

```toml
[profile.release]
opt-level = 3
lto = true
codegen-units = 1
```

Целевой triple требует установленного toolchain и linker. Для кросс-компиляции проверьте платформенные зависимости.

## Тесты и документация

Сохраните следующий код в `src/lib.rs`.

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn adds_numbers() {
        assert_eq!(add(2, 3), 5);
    }
}
```

```bash
cargo test adds_numbers
cargo test --doc
cargo doc --no-deps --open
```

## Рабочий процесс

```bash
cargo fmt --all -- --check
cargo check --all-targets --all-features
cargo test --all-features
cargo clippy --all-targets --all-features -- -D warnings
```

`-D warnings` превращает предупреждения Clippy в ошибки. Используйте его в CI после согласования набора lint с проектом.

## Публикация

```bash
cargo login
cargo package --list
cargo publish --dry-run
cargo publish
```

Токен Cargo хранится локально и не должен попадать в Git или логи CI. Перед публикацией проверьте имя, лицензию, описание, README, включённые файлы и номер версии.

## Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `could not find Cargo.toml` | Команда запущена вне пакета | Перейдите в каталог проекта или используйте `--manifest-path` |
| `failed to select a version` | Несовместимые требования зависимостей | Проверьте `cargo tree`, версии и feature flags |
| `linker cc not found` | Нет системного компилятора или linker | Установите toolchain ОС и проверьте `rustc -vV` |
| `target ... may not be installed` | Нет целевого Rust toolchain | Установите target через `rustup target add` |
| `cargo publish` отклонён | Не заполнены метаданные или имя занято | Проверьте `cargo package --list` и `Cargo.toml` |

## Документация

- [Cargo Book](https://doc.rust-lang.org/cargo/)
- [Cargo command reference](https://doc.rust-lang.org/cargo/commands/index.html)
- [Rustup](https://rust-lang.github.io/rustup/)
- [crates.io](https://crates.io/)
- [Основы Rust](../programming/Rust.md)

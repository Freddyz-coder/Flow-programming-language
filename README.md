# Flow-programming-language
everything is inside the flow.tar
its a custom programming language
its not ready for deployment althought you can help us grow the flow language bigger

# for windows 11
Install Rust (if needed)

https://rustup.rs → run the installer, then open a new terminal.

Go to the project bat cd path\to\flow

Build bat cargo build(or cargo build --release for a faster binary)

Run an example bat

cargo run -- examples\fizzbuzz.flow 
directly:bat.\target\debug\flow.exe examples\fizzbuzz.flow

That’s it. Replace fizzbuzz.flow with any file in examples\.

# On macOS

Install Rust (if needed):

Bashcurl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

Go to the project and build:

Bashcd flow
cargo build --release

Run any example:

Bash./target/release/flow examples/fizzbuzz.flow
Or without the full path after cargo run:
Bash cargo run -- examples/fizzbuzz.flow
That’s it.

# on linux
bash
cd /path/to/flow
cargo build
./target/debug/flow examples/fizzbuzz.flow

Or in one step:
Bashcargo run -- examples/fizzbuzz.flow

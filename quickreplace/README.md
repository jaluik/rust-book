# QuickReplace

QuickReplace is a simple Rust project that allows you to quickly replace text in a file. This tool is useful for making bulk text replacements in a file with ease.

## Usage

To use QuickReplace, run the following command:

```sh
./target/release/quickreplace <file_path> <search_text> <replace_text>
```

- `<file_path>`: The path to the file where you want to perform the replacement.
- `<search_text>`: The text you want to search for.
- `<replace_text>`: The text you want to replace the search text with.

## Example

Suppose you have a file `example.txt` with the following content:

```
Hello, world!
Hello, Rust!
```

To replace "Hello" with "Hi", run the following command:

```sh
./target/release/quickreplace example.txt Hello Hi
```

After running the command, the content of `example.txt` will be:

```
Hi, world!
Hi, Rust!
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request if you have any improvements or bug fixes.

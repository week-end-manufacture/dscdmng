# dscdmng

A small C command-line program that sends text messages through a Discord webhook. It can post notifications to a Discord channel using only a webhook URL and a message, without requiring a separate Discord bot or a resident process.

## Features

- Sends text messages through the Discord Webhook API
- Sends HTTPS POST requests using `libcurl`
- Builds a JSON request body and escapes special characters
- Applies a 10-second request timeout
- Treats HTTP `200` and `204` responses as successful
- Retries HTTP failures up to five times
- Prints the result and HTTP status without printing the response body
- Supports system-wide installation with `make install`

## How It Works

The program follows these steps:

1. Reads the webhook URL and message from the command-line arguments.
2. Escapes double quotes, backslashes, newlines, carriage returns, and tabs in the message for use in a JSON string.
3. Builds a JSON request body in the following format:

	```json
	{"content": "Message to send"}
	```

4. Sends a POST request to the webhook URL with the `Content-Type: application/json` header.
5. Uses a 10-second timeout and considers HTTP `200` or `204` a success.
6. On an HTTP error, waits one second and retries the same request up to five times.

The response body is not stored or printed. As a result, errors are reported using a libcurl error message or HTTP status code rather than the detailed error body returned by the Discord API.

## Requirements

- A C compiler (`gcc` is recommended)
- The `libcurl` development library and headers
- A network connection
- A webhook URL for the target Discord channel

On macOS, `libcurl` can be installed with Homebrew:

```sh
brew install curl
```

If it is installed outside the default system paths, you may need to set `CPPFLAGS` and `LDFLAGS` so the compiler can find its headers and libraries.

## Build

From the repository root, change to the `src` directory and run `make`:

```sh
cd src
make
```

The command creates the `src/dscd` executable. The default Makefile settings are:

- Compiler: `gcc`
- Warning flags: `-Wall -Wextra`
- Optimization: `-O2`
- Linked library: `-lcurl`

You can also build the program directly without Make:

```sh
cd src
gcc -Wall -Wextra -O2 -o dscd main.c -lcurl
```

## Set Up a Discord Webhook

1. Open the Discord server and channel where messages should be posted.
2. Open the channel settings and select **Integrations**.
3. Create a **Webhook**.
4. Copy the generated webhook URL.

A webhook URL usually has this form:

```text
https://discord.com/api/webhooks/<webhook-id>/<webhook-token>
```

The webhook URL contains the token required for authentication. Keep it out of source code, public repositories, logs, and screen shares. Store it in an environment variable or a secret store. If the URL is exposed, regenerate or delete the webhook in Discord.

## Usage

```sh
./dscd "<WEBHOOK_URL>" "<MESSAGE>"
```

Example:

```sh
./dscd "https://discord.com/api/webhooks/123456789/abcdef" "Server maintenance is complete."
```

Quoting the URL and message is recommended because messages may contain spaces or characters interpreted specially by the shell.

Multiline messages can also be sent:

```sh
./dscd "$DISCORD_WEBHOOK_URL" $'Deployment complete\nVersion: 1.2.3'
```

Example using an environment variable for the webhook URL:

```sh
export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/<webhook-id>/<webhook-token>"
./dscd "$DISCORD_WEBHOOK_URL" "The backup job is complete."
```

## Install and Uninstall

The executable can be installed as `/usr/local/bin/dscd`:

```sh
cd src
make install
```

If you do not have write permission for `/usr/local/bin`, run the installation with administrator privileges:

```sh
sudo make install
```

Remove the installed executable with:

```sh
sudo make uninstall
```

## Output and Exit Status

If the arguments are invalid, the program prints the usage syntax and an example to standard error, then exits with a failure status.

On success, it prints a message such as:

```text
Message sent successfully (HTTP 204)
```

Possible error output includes:

```text
Transfer failed: Couldn't resolve host
Transfer failed (HTTP 401) - check the webhook URL or message format.
Retrying... (5 attempts remaining)
Retry 1 failed (HTTP 401)
```

Repeated `HTTP 401` or `HTTP 404` responses usually indicate that the webhook URL is invalid or has been deleted. For network or DNS errors, check the network connection and URL before trying again.

## Project Structure

```text
.
├── README.md          Project documentation and usage instructions
└── src/
	 ├── main.c         Webhook request, transmission, and retry logic
	 ├── makefile       Build, installation, and removal rules
	 ├── define.h       Common definitions header
	 ├── extern.h       External declarations header
	 └── global.h       Global declarations header
```

The active transmission logic is currently implemented in `main.c`. The three header files provide a basic structure for adding shared declarations later.

## Clean Build Artifacts

```sh
cd src
make clean
```

`make clean` removes only the `src/dscd` binary and does not delete source files.

## Limitations

- The program currently sends messages to one Discord webhook per invocation.
- Because the message is passed as a command-line argument, very long messages may be limited by shell or operating-system argument length limits.
- Additional Discord webhook options such as embeds, usernames, avatars, and file attachments are not supported.
- Because the response body is discarded, detailed Discord API error reasons are not available in the output.
- TLS certificate verification remains enabled by default. Disabling certificate verification is not recommended for security reasons.

## License

See the repository's [LICENSE](LICENSE) file for license information.
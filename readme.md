# Gevent Key-Value Store

This is a lightweight, in-memory key-value database implemented in Python using `gevent`. It features a custom TCP protocol inspired by RESP (Redis Serialization Protocol) and includes both server and client implementations within a single file.


## Dependencies

  * Python 2.7+ or 3.x
  * `gevent`

To install the necessary library:

```bash
pip install gevent
```

## Architecture

The project consists of three main components:

1.  **ProtocolHandler:** Manages the serialization and deserialization of data sent over the socket. It supports simple strings, errors, integers, bulk strings, arrays, and dictionaries.
2.  **Server:** A `StreamServer` implementation that listens on a specified port, accepts connections, and processes commands against an in-memory dictionary. It uses a `gevent.pool.Pool` to manage concurrent client connections.
3.  **Client:** A wrapper around a TCP socket that simplifies sending commands and receiving parsed responses from the server.

## Usage

### Starting the Server

You can run the script directly to start the server. By default, it listens on `127.0.0.1:31337`.

```bash
python kv_store.py
```

*Note: The script attempts to patch standard library modules using `gevent.monkey.patch_all()` in the `__main__` block to enable asynchronous I/O.*

### Using the Client

You can import the `Client` class in a separate Python script or an interactive shell to interact with the running server.

```python
from kv_store import Client

# Connect to the server
client = Client()

# Set a value
client.set('username', 'jdoe')

# Get a value
print(client.get('username'))  # Output: jdoe

# Set multiple values
client.mset('a', 1, 'b', 2, 'c', 3)

# Get multiple values
print(client.mget('a', 'b'))  # Output: [1, 2]

# Delete a value
client.delete('username')
```

## Supported Commands

The server currently implements the following commands:

  * **GET \<key\>**: Retrieves the value associated with the key.
  * **SET \<key\> \<value\>**: Stores a value with the given key.
  * **DELETE \<key\>**: Removes the key from the store.
  * **FLUSH**: Clears all data from the store.
  * **MGET \<key1\> \<key2\> ...**: Retrieves values for multiple keys at once.
  * **MSET \<key1\> \<val1\> \<key2\> \<val2\> ...**: Sets multiple key-value pairs in a single operation.

## Protocol Details

The `ProtocolHandler` uses a prefix-based format to identify data types, similar to Redis:

  * `+`: Simple String
  * `-`: Error
  * `:`: Integer
  * `$`: Bulk String (followed by length)
  * `*`: Array (followed by element count)
  * `%`: Dictionary (followed by item count)

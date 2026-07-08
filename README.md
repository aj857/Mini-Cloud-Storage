# Mini Cloud Storage System in C

A lightweight, multithreaded cloud storage system built in **C** using **TCP socket programming** and a **client-server architecture**.

The project enables clients to connect to a storage server and remotely **upload, download, list, rename, and delete files** through an interactive command-line interface. It includes a POSIX implementation for Linux/macOS-compatible environments and separate Windows implementations using Winsock.

## Overview

This project demonstrates core systems programming and computer networking concepts by implementing a custom file storage service from scratch.

The server listens for incoming TCP connections, creates a separate thread for each connected client, processes file-management commands, and stores files inside a configurable storage directory. The client provides an interactive shell for communicating with the server.

## Key Features

- TCP-based client-server communication
- Multithreaded server for handling multiple clients
- File upload from client to server
- File download from server to client
- Remote file listing with file sizes
- Remote file rename operation
- Remote file deletion
- Configurable server storage directory
- Custom application-layer command protocol
- File locking for concurrent access on POSIX systems
- Path traversal protection for remote filenames
- Efficient buffered file transfer
- Linux `sendfile()` optimization for downloads
- Graceful client disconnect using the `QUIT` command
- Cross-platform source code for POSIX and Windows environments

## Tech Stack

| Component | Technology |
|---|---|
| Language | C |
| Networking | TCP/IP, BSD Sockets |
| Concurrency | POSIX Threads (`pthread`) |
| File Synchronization | `fcntl()` Advisory Locks |
| Linux Transfer Optimization | `sendfile()` |
| Windows Networking | Winsock2 |
| Windows Concurrency | Win32 Threads |
| Build System | Make |
| Compiler | GCC |

## System Architecture

```text
                    TCP/IP Network
        ┌──────────────────────────────────┐
        │                                  │
┌───────▼────────┐                ┌────────▼─────────┐
│     Client     │                │      Server      │
│                │                │                  │
│ Interactive    │   Commands     │ TCP Listener     │
│ CLI / REPL     ├───────────────►│                  │
│                │                │ Thread per       │
│ Upload         │   Responses    │ Client           │
│ Download       │◄───────────────┤                  │
│ List           │                │ File Operations  │
│ Rename         │                │                  │
│ Delete         │                │ Storage Directory│
└────────────────┘                └────────┬─────────┘
                                         │
                                         ▼
                                ┌──────────────────┐
                                │     storage/     │
                                │                  │
                                │  Uploaded Files  │
                                └──────────────────┘
```

## Project Structure

```text
.
├── client.c
├── server.c
├── client_windows.c
├── server_windows.c
├── Makefile
└── README.md
```

### File Description

| File | Description |
|---|---|
| `server.c` | POSIX TCP server with multithreading, file locking, and file operations |
| `client.c` | POSIX command-line client |
| `server_windows.c` | Windows server implementation using Winsock2 and Win32 threads |
| `client_windows.c` | Windows client implementation using Winsock2 |
| `Makefile` | Build configuration for POSIX server and client |

## Supported Commands

After connecting to the server, the client displays:

```text
cloud>
```

The following commands are supported:

| Command | Description |
|---|---|
| `list` | List remotely stored files and their sizes |
| `upload <localpath>` | Upload a local file using its original filename |
| `upload <localpath> <remote_name>` | Upload a file with a custom remote filename |
| `download <remote_name>` | Download a remote file |
| `download <remote_name> <save_as>` | Download and save using a custom local filename |
| `rename <oldname> <newname>` | Rename a remote file |
| `delete <remote_name>` | Delete a remote file |
| `quit` | Close the client connection gracefully |

## Custom Application Protocol

The project uses a simple text-based command protocol over TCP.

### Client-to-Server Commands

```text
LIST
UPLOAD <filename> <size>
DOWNLOAD <filename>
RENAME <oldname> <newname>
DELETE <filename>
QUIT
```

### Server Responses

Successful operations use responses beginning with:

```text
OK
```

Errors use responses beginning with:

```text
ERR <message>
```

For file transfers, control messages are exchanged as text while file contents are transmitted as raw bytes over the same TCP connection.

## Build and Run

### Prerequisites

For the POSIX version:

- GCC
- Make
- POSIX threads
- Linux, Ubuntu, WSL, or another compatible POSIX environment

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd <your-repository-name>
```

### 2. Build the Project

```bash
make
```

This creates two executables:

```text
server
client
```

### 3. Start the Server

Open the first terminal:

```bash
./server 8080 storage
```

Expected output:

```text
Server listening on port 8080, storage: storage
```

The arguments are:

```text
./server <port> [storage_directory]
```

Example:

```bash
./server 8080 storage
```

If no storage directory is provided, the server uses:

```text
storage
```

### 4. Start the Client

Open a second terminal:

```bash
./client 127.0.0.1 8080
```

Expected output:

```text
OK WELCOME
cloud>
```

The arguments are:

```text
./client <server_ip> <port>
```

For local testing:

```bash
./client 127.0.0.1 8080
```

## Usage Examples

### List Stored Files

```text
cloud> list
```

Example output:

```text
Files (2):
  report.pdf                     204800 bytes
  notes.txt                      1024 bytes
```

### Upload a File

```text
cloud> upload notes.txt
```

Upload with a custom remote filename:

```text
cloud> upload notes.txt cloud_notes.txt
```

### Download a File

```text
cloud> download cloud_notes.txt
```

Download with a custom local filename:

```text
cloud> download cloud_notes.txt downloaded_notes.txt
```

### Rename a File

```text
cloud> rename cloud_notes.txt final_notes.txt
```

### Delete a File

```text
cloud> delete final_notes.txt
```

### Exit the Client

```text
cloud> quit
```

## Concurrency Model

The POSIX server uses a **thread-per-client architecture**.

```text
                    ┌──────────────┐
                    │ TCP Server   │
                    │ Listening    │
                    │ Socket       │
                    └──────┬───────┘
                           │
                         accept()
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
        ┌────────┐    ┌────────┐    ┌────────┐
        │Thread 1│    │Thread 2│    │Thread 3│
        │Client A│    │Client B│    │Client C│
        └────────┘    └────────┘    └────────┘
```

Each accepted client connection is assigned to a separate worker thread, allowing multiple clients to interact with the server concurrently.

## File Synchronization

The POSIX implementation uses `fcntl()` advisory file locks to reduce conflicts during concurrent file access.

- Upload operations use an exclusive write lock
- Download operations use a shared read lock
- Rename operations use a write lock
- Delete operations attempt a write lock before deletion

This demonstrates synchronization between concurrent file operations.

## Security Considerations

The server validates remote filenames before constructing storage paths.

Filenames containing the following are rejected:

```text
..
/
\
```

This helps reduce directory traversal attempts and prevents clients from directly escaping the configured storage directory through remote filenames.

> This project is intended for educational and systems-programming purposes. It does not currently provide production-grade authentication, encryption, access control, or sandboxing.

## Performance Considerations

The project includes several implementation choices aimed at reliable and efficient transfer:

- Repeated `send()` operations to handle partial socket writes
- Repeated `recv()` operations to receive exact byte counts
- Buffered file transfer using 64 KiB buffers
- Linux `sendfile()` support for efficient server-side downloads
- Detached worker threads for concurrent clients
- TCP socket reuse through `SO_REUSEADDR`

## Cross-Platform Support

### POSIX Version

Uses:

- BSD/POSIX sockets
- `pthread`
- `fcntl`
- POSIX file APIs
- `sendfile()` on Linux

Source files:

```text
server.c
client.c
```

### Windows Version

Uses:

- Winsock2
- Win32 file APIs
- Win32 threads
- `CreateFile()`
- `ReadFile()`
- `WriteFile()`

Source files:

```text
server_windows.c
client_windows.c
```

## Clean Build Files

To remove generated executables:

```bash
make clean
```

## Learning Outcomes

This project demonstrates practical understanding of:

- TCP socket programming
- Client-server architecture
- Application-layer protocol design
- Concurrent server development
- POSIX threads
- File descriptors
- File I/O
- File synchronization and locking
- Binary data transfer over TCP
- Error handling in systems programming
- Cross-platform networking concepts
- Basic path validation and security considerations

## Future Improvements

Potential extensions include:

- User authentication and authorization
- TLS-encrypted communication
- Per-user storage directories
- File integrity verification using SHA-256
- Upload and download progress indicators
- Resumable file transfers
- Storage quotas
- File versioning
- Metadata persistence
- Structured logging
- Automated unit and integration tests
- Docker-based deployment
- Web or desktop client interface

## Author

**Arjunsinh Vaghela**

Information and Communication Technology  
Dhirubhai Ambani University, Gandhinagar

## License

This project is intended for educational and portfolio purposes.

If you plan to distribute or accept external contributions, consider adding an open-source license such as the MIT License.

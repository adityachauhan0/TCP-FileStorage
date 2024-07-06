# Distributed File Storage System

## Overview

This project implements a decentralized, fully distributed content-addressable file storage system using Go. It is designed to handle and stream very large files efficiently.

## Features

- **Decentralized Architecture**: Utilizes a peer-to-peer network for distributed file storage.
- **Streaming Capabilities**: Supports streaming of large files.
- **Encryption**: Files are encrypted for secure storage and retrieval.
- **Fault Tolerance**: Designed to handle node failures gracefully.
- **Scalability**: Can scale by adding more nodes to the network.

## Components

### Components Included:

- **main.go**: Entry point of the application, sets up and starts file servers.
- **server.go**: Implements the file server functionality.
- **store.go**: Handles storage operations, including encryption and path transformations.
- **p2p/**: Contains supporting peer-to-peer networking functionalities.

## Installation

### Prerequisites

- Go 1.16 or higher installed.
- Ensure GOPATH and GOROOT environment variables are correctly set.

### Setup Instructions

1. Clone the repository:

   ```
   git clone <repository_url>
   cd distributed-file-system
   ```

2. Install dependencies:

   ```
   go mod tidy
   ```

3. Build the application:
   ```
   go build
   ```

## Usage

### Running the Application

1. Start individual servers:

   ```
   ./distributed-file-system :3000
   ./distributed-file-system :7000
   ```

2. Start a server with bootstrap nodes:

   ```
   ./distributed-file-system :5000 :3000 :7000
   ```

3. The servers will automatically bootstrap and start accepting requests.

## Example

The `main.go` file contains an example of how to set up and use the file servers. It demonstrates storing and retrieving files from the network.

```go
package main

import (
	"bytes"
	"fmt"
	"io/ioutil"
	"log"
	"time"

	"github.com/adityachauhan0/store/p2p"
)

func main() {
	// Create servers
	s1 := makeServer(":3000", "")
	s2 := makeServer(":7000", "")
	s3 := makeServer(":5000", ":3000", ":7000")

	// Start servers concurrently
	go func() {
		if err := s1.Start(); err != nil {
			log.Fatalf("Failed to start server 1: %v", err)
		}
	}()
	time.Sleep(500 * time.Millisecond)

	go func() {
		if err := s2.Start(); err != nil {
			log.Fatalf("Failed to start server 2: %v", err)
		}
	}()
	time.Sleep(500 * time.Millisecond)

	go func() {
		if err := s3.Start(); err != nil {
			log.Fatalf("Failed to start server 3: %v", err)
		}
	}()

	time.Sleep(2 * time.Second)

	// Example usage: storing and retrieving files
	for i := 0; i < 3; i++ {
		key := fmt.Sprintf("picture_%d.png", i)
		data := bytes.NewReader([]byte("my big data file here!"))

		// Store a file
		s3.Store(key, data)

		// Retrieve the file
		r, err := s3.Get(key)
		if err != nil {
			log.Fatalf("Failed to retrieve file %s: %v", key, err)
		}

		// Read the retrieved file content
		b, err := ioutil.ReadAll(r)
		if err != nil {
			log.Fatalf("Failed to read file content %s: %v", key, err)
		}

		// Print the file content
		fmt.Printf("Content of %s:\n%s\n", key, string(b))

		// Delete the stored file
		if err := s3.Delete(key); err != nil {
			log.Fatalf("Failed to delete file %s: %v", key, err)
		}
	}
}

```

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your improvements.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

Feel free to customize the sections further based on specific details or additional functionalities of your project.

# PlexLabelarr 🎬🏷️

[![Docker Image](https://img.shields.io/docker/v/nullableeth/plexlabelarr?style=flat-square)](https://hub.docker.com/r/nullableeth/plexlabelarr)
[![Docker Pulls](https://img.shields.io/docker/pulls/nullableeth/plexlabelarr?style=flat-square)](https://hub.docker.com/r/nullableeth/plexlabelarr)
[![Go Version](https://img.shields.io/github/go-mod/go-version/nullable-eth/PlexLabelarr?style=flat-square)](https://golang.org/)

Automatically sync TMDb movie keywords as Plex labels - A lightweight Go application that bridges your Plex movie library with The Movie Database, adding relevant keywords as searchable labels.

## 🚀 Features

- 🔄 **Periodic Processing**: Automatically processes movies on a configurable timer
- 🏷️ **Smart Label Management**: Adds TMDb keywords as Plex labels without removing existing labels
- 🔍 **Flexible TMDb ID Detection**: Extracts TMDb IDs from Plex metadata or file paths
- 📊 **Progress Tracking**: Maintains a dictionary of processed movies to avoid duplicates
- 🐳 **Docker Ready**: Containerized for easy deployment
- ⚙️ **Environment Configuration**: Fully configurable via environment variables
- 🔒 **Protocol Flexibility**: Supports both HTTP and HTTPS Plex connections
- Allows you to have TMBDB keywords as labels in Plex:
![Screenshot 2025-06-12 100834](https://github.com/user-attachments/assets/04a01fee-0351-47c3-862c-0e71cfca2456)
- Create custom dynamic filters for multiple labels that will update automatically when new movies are labeled:
![Screenshot 2025-06-12 101530](https://github.com/user-attachments/assets/54a1686e-2a29-4958-9e6e-1a592a837130)
- Filter on the fly by a label:

![Screenshot 2025-06-12 101413](https://github.com/user-attachments/assets/2b35580f-6443-4a6d-9e40-f8217e660699)


## 📋 Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `PLEX_SERVER` | Plex server IP/hostname | - | **Yes** |
| `PLEX_PORT` | Plex server port | - | **Yes** |
| `PLEX_REQUIRES_HTTPS` | Use HTTPS for Plex connection | `true` | No |
| `PLEX_TOKEN` | Plex authentication token | - | **Yes** |
| `TMDB_API_KEY` | TMDb API Bearer token | - | **Yes** |
| `PROCESS_TIMER` | Processing interval (e.g., `5m`, `1h`) | `5m` | No |
| `LIBRARY_ID` | Plex library ID (auto-detected if not set) | - | No |

## 🔑 Getting API Keys

### Plex Token

1. Open Plex Web App in browser
2. Press F12 → Network tab
3. Refresh the page
4. Find any request with `X-Plex-Token` in headers
5. Copy the token value

### TMDb API Key

1. Visit [TMDb API Settings](https://www.themoviedb.org/settings/api)
2. Create account and generate API key
3. Use the **Bearer Token** (not the API key)

## 🐳 Docker Deployment

### Quick Start

```bash
docker run -d --name plexlabelarr \
  -e PLEX_SERVER=192.168.1.12 \
  -e PLEX_PORT=32400 \
  -e PLEX_REQUIRES_HTTPS=true \
  -e PLEX_TOKEN=your_plex_token_here \
  -e TMDB_API_KEY=your_tmdb_bearer_token_here \
  -e PROCESS_TIMER=5m \
  nullableeth/plexlabelarr:latest
```

### Docker Compose

1. Download the `docker-compose.yml` file from this repository
2. Update environment variables with your credentials:

```yaml
version: '3.8'

services:
  plexlabelarr:
    image: nullableeth/plexlabelarr:latest
    container_name: plexlabelarr
    restart: unless-stopped
    environment:
      - PLEX_SERVER=192.168.1.12
      - PLEX_PORT=32400
      - PLEX_REQUIRES_HTTPS=true
      - PLEX_TOKEN=your_plex_token_here
      - TMDB_API_KEY=your_tmdb_api_key_here
      - PROCESS_TIMER=5m
    networks:
      - plexlabelarr
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  plexlabelarr:
    driver: bridge
```

3. Run: `docker-compose up -d`

## 🛠️ Local Development

### Prerequisites

- Go 1.23+
- Git

### Build and Run

```bash
# Clone the repository
git clone https://github.com/nullable-eth/PlexLabelarr.git
cd PlexLabelarr

# Initialize Go modules
go mod tidy

# Set environment variables
export PLEX_SERVER=192.168.1.12
export PLEX_PORT=32400
export PLEX_TOKEN=your_plex_token
export TMDB_API_KEY=your_tmdb_api_key

# Run the application
go run main.go
```

### Build Binary

```bash
# Build for current platform
go build -o plexlabelarr main.go

# Build for Linux (Docker)
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o plexlabelarr main.go
```

## 📖 How It Works

1. **Library Discovery**: Automatically finds your movie library
2. **Movie Processing**: Iterates through all movies in the library
3. **TMDb ID Extraction**: Gets TMDb IDs from:
   - Plex metadata Guid field
   - File/folder names with `{tmdb-12345}` format
4. **Keyword Fetching**: Retrieves keywords from TMDb API
5. **Label Synchronization**: Adds new keywords as labels (preserves existing labels)
6. **Progress Tracking**: Remembers processed movies to avoid re-processing

## 🔍 TMDb ID Detection

The application can find TMDb IDs from multiple sources:

- **Plex Metadata**: Standard TMDb agent IDs
- **File Paths**: `{tmdb-12345}` in filenames or directory names
- **Various Formats**: Supports different TMDb ID formats

Example file paths:

```
/data/Movies/Zeitgeist - Moving Forward (2011) {tmdb-54293}/movie.mp4
/movies/The Matrix (1999) {tmdb-603}/The Matrix.mkv
```

## 📊 Monitoring

### View Logs

```bash
# Docker logs
docker logs plexlabelarr

# Follow logs
docker logs -f plexlabelarr
```

### Log Output Includes

- Processing progress with movie counts
- TMDb ID detection results
- Label synchronization status
- API error handling and retries
- Detailed processing summaries

## ⚙️ Configuration Examples

### For HTTP-only Plex servers

```bash
-e PLEX_REQUIRES_HTTPS=false
```

### For frequent processing

```bash
-e PROCESS_TIMER=2m
```

### For specific library

```bash
-e LIBRARY_ID=1
```

## 🔧 Troubleshooting

### Common Issues

**401 Unauthorized from Plex**

- Verify your Plex token is correct
- Check if your Plex server requires HTTPS

**401 Unauthorized from TMDb**

- Ensure you're using the Bearer token, not API key
- Verify the token has proper permissions

**No TMDb ID found**

- Check if your movies have TMDb metadata
- Verify file naming includes `{tmdb-12345}` format
- Ensure TMDb agent is used in Plex

**Connection refused**

- Check PLEX_SERVER and PLEX_PORT values
- Try setting PLEX_REQUIRES_HTTPS=false for local servers

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Plex](https://www.plex.tv/) for the amazing media server
- [The Movie Database (TMDb)](https://www.themoviedb.org/) for the comprehensive movie data
- [Go](https://golang.org/) for the excellent programming language

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/nullable-eth/PlexLabelarr/issues)
- **Docker Hub**: [nullableeth/plexlabelarr](https://hub.docker.com/r/nullableeth/plexlabelarr)
- **Documentation**: This README and inline code comments

---

⭐ **If you find this project helpful, please consider giving it a star!**
